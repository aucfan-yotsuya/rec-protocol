新卒講義:プロトコル 2020-07-07(Tue)
==


## Docker環境について
- 必須ではありませんが、Linux環境必要であればお使いください。
    - 要: docker / docker-compose

```
% cd docker
# 起動
% docker-compose up --build -d
# シェル実行
% docker-compose exec al2 bash
# 削除
% docker-compose down --rmi all
```

---

## TCP echo serverサンプル

```
bash-4.2# cd echoserver/
bash-4.2# ls -la
total 8
drwxr-xr-x 4 root root  128 Jun 30 18:37 .
drwxr-xr-x 9 root root  288 Jul  3 13:48 ..
-rw-r--r-- 1 root root 1483 Jun 25 16:08 client.c
-rw-r--r-- 1 root root 1747 Jun 25 16:02 server.c
bash-4.2# gcc -Wall -o client client.c
bash-4.2# gcc -Wall -o server server.c
bash-4.2# ./server
Server running...waiting for connections.
^C
bash-4.2# ./server &
[1] 25
bash-4.2# Server running...waiting for connections.
./client 127.0.0.1
Received request...
Child created for dealing with client requests
hogehoge
String received from and resent to the client:hogehoge


String received from the server: hogehoge
bash-4.2#
```

---

## 課題: CLUDサーバー/クライアント開発

- Redisにkey/valueをset/get/delするclient -> serverを開発。

### server仕様
- tcp/4280 でlisten
- 待受メソッド
    - "W" (書き込みモード)
    - "R" (読み込みモード)
    - "D" (削除モード)

### シーケンス

```
[client] <----> [server] <--(RESP)--> [redis]
```

### 通信仕様

- "W" (書き込みモード)
    - 入力されたkey/valueをRedisにsetする
        - 存在するkeyは上書きとする。
        - Redis側のkeyの型はString型とする。
    - client -> server リクエスト仕様
        - "W"(0x57), 0x00, KEY, 0x00, VALUE, 0x00
        - 例) KEY="FOO"、VALUE="BAR"を作成する電文
        ```
        % echo -n "W\x00FOO\x00BAR\x00" | xxd -u
        00000000: 5700 464F 4F00 4241 5200                 W.FOO.BAR.
    - server -> client レスポンス仕様
        - 正常時
            - 0x00
        - 異常時
            - 0xFF
- "R" (読み込みモード)
    - 入力されたkeyをRedisからgetしvalueを応答する
        - keyが存在しない場合は異常時のレスポンスとする
    - client -> server リクエスト仕様
        - "R"(0x52), 0x00, KEY, 0x00
        - 例) KEY="FOO"を削除する電文
        ```
        % echo -n "R\x00FOO\x00" | xxd -u
        00000000: 5200 464F 4F00                           R.FOO.
    - server -> client レスポンス仕様
        - 正常時
            - 0x00, VALUE, 0x00
            - 例) KEY="FOO", VALUE="BAR"の場合
            ```
            % echo -n "\x00BAR\x00" | xxd -u
            00000000: 0042 4152 00                             .BAR.
            ```
        - 異常時
            - 0xFF
- "D" (削除モード)
    - 入力されたkeyをRedisからdelする
        - keyが存在しない場合は異常時のレスポンスとする
    - client -> server リクエスト仕様
        - "D"(0x44), 0x00, KEY, 0x00
        - 例) KEY="FOO"を削除する電文
        ```
        % echo -n "D\x00FOO\x00" | xxd -u
        00000000: 4400 464F 4F00                           D.FOO.
        ```
    - server -> client レスポンス仕様
        - 正常時
            - 0x00
        - 異常時
            - 0xFF
- 上記に該当しないモード指定
    - すべて異常応答とする
        - 0xFF

### client動作仕様
- CLI
    - パラメーター
        - IPアドレス(4octet)
        - モード(W/R/D)
        - KEY(string)
        - VALUE(string)
    - 実行結果
        - 全モード共通
            - 正常時
                - "SUCESS" と表示
            - 異常時
                - "FAIL" と表示
    - 実行例

    ```
    % ./server &
    % ./client 192.168.0.1 W hello world
    SUCCESS
    % redis-cli -h x.x.x.x
    x.x.x.x:6379> get hello
    "world"
    ```
