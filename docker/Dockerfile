FROM amazonlinux:2

RUN yum update -y
RUN yum install systemd-sysv -y
RUN amazon-linux-extras install redis4.0 -y
RUN systemctl enable redis.service
RUN yum install tcpdump -y
RUN yum groupinstall "Development Tools" -y
RUN yum install git.x86_64 -y
RUN yum install golang.x86_64 -y

# Go env
RUN echo 'export GOPATH=$HOME/go' >> ~/.bashrc
RUN echo 'PATH="$PATH:$GOPATH/bin"' >> ~/.bashrc
RUN ln -sf  /usr/share/zoneinfo/Asia/Tokyo /etc/localtime
# 
CMD ["/sbin/init"]
