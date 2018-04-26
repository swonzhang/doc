
安装docker: 

refer: https://www.cnblogs.com/yufeng218/p/8370670.html

yum install docker


安装docker镜像
refer: https://blog.csdn.net/S_gy_Zetrov/article/details/78161154


1\  docker search centos

2\ docker pull centos:latest

3\ docker images

4\ docker run -it centos:latest /bin/bash

error: /usr/bin/docker-current: Error response from daemon: error creating overlay mount to /var/lib/docker/overlay2/ae0588ed0ec5a44efadf5adcb29b7be9e1313983850799016780940c71a9aef3/merged: invalid argument.


then refer : https://blog.csdn.net/Ysssssssssssssss/article/details/79596367

ok,go on

4\ docker run -it centos /bin/bash

ok ,install

ctrl+P and ctrl+Q  quit the docker

then,into it

5\ docker exec -it containerID  bash

Dockerfile

FROM docker.listcloud.cn:5000/nginx-php7

COPY . /var/www/html

COPY ./devops/config/default.conf /etc/nginx/nginx.conf

COPY ./devops/config/vhost.conf /etc/nginx/conf.d/vhost.conf

COPY ./devops/config/mime.types /etc/nginx/mime.types

WORKDIR /var/www/html


