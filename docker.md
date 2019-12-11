
安装docker: 

refer: https://www.cnblogs.com/yufeng218/p/8370670.html

yum install docker


安装docker镜像
refer: https://blog.csdn.net/S_gy_Zetrov/article/details/78161154



注意：docker默认使用国外镜像，由于被墙，所以需要做个备用请求

vim /etc/docker/daemon.json

添加

{
	"registry-mirrors":["https://registry.docker-cn.com"],
  "insecure-registries": ["docker.ops.colourlife.com:5000","docker.listcloud.cn:5000"]
}


下面可以安装镜像


1\  docker search centos

2\ docker pull centos:latest

3\ docker images

4\ docker run -it centos:latest /bin/bash

error: /usr/bin/docker-current: Error response from daemon: error creating overlay mount to /var/lib/docker/overlay2/ae0588ed0ec5a44efadf5adcb29b7be9e1313983850799016780940c71a9aef3/merged: invalid argument.

出现上面错误，可以参考 修改 /etc/sysconfig 下的docker文件
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


编辑好之后，在相同目录下运行：docker build -t swon/nginx-php7 .

-t 后面跟镜像标签名称

注意后面那一点，表示是当前目录，建立后之后，docekr images,即可看到自己打包的镜像 

有时候需要挂载docker的文件在宿主机

-v /var/www/html/data:/data  

表示该命令将挂载主机的/var/www/html/data目录到容器内的/data目录上

Usage: docker run [OPTIONS] IMAGE [COMMAND] [ARG...]    
  
  -d, --detach=false         指定容器运行于前台还是后台，默认为false     
  -i, --interactive=false   打开STDIN，用于控制台交互    
  -t, --tty=false            分配tty设备，该可以支持终端登录，默认为false    
  -u, --user=""              指定容器的用户    
  -a, --attach=[]            登录容器（必须是以docker run -d启动的容器）  
  -w, --workdir=""           指定容器的工作目录   
  -c, --cpu-shares=0        设置容器CPU权重，在CPU共享场景使用    
  -e, --env=[]               指定环境变量，容器中可以使用该环境变量    
  -m, --memory=""            指定容器的内存上限    
  -P, --publish-all=false    指定容器暴露的端口    
  -p, --publish=[]           指定容器暴露的端口   
  -h, --hostname=""          指定容器的主机名    
  -v, --volume=[]            给容器挂载存储卷，挂载到容器的某个目录    
  --volumes-from=[]          给容器挂载其他容器上的卷，挂载到容器的某个目录  
  --cap-add=[]               添加权限，权限清单详见：http://linux.die.net/man/7/capabilities    
  --cap-drop=[]              删除权限，权限清单详见：http://linux.die.net/man/7/capabilities    
  --cidfile=""               运行容器后，在指定文件中写入容器PID值，一种典型的监控系统用法    
  --cpuset=""                设置容器可以使用哪些CPU，此参数可以用来容器独占CPU    
  --device=[]                添加主机设备给容器，相当于设备直通    
  --dns=[]                   指定容器的dns服务器    
  --dns-search=[]            指定容器的dns搜索域名，写入到容器的/etc/resolv.conf文件    
  --entrypoint=""            覆盖image的入口点    
  --env-file=[]              指定环境变量文件，文件格式为每行一个环境变量    
  --expose=[]                指定容器暴露的端口，即修改镜像的暴露端口    
  --link=[]                  指定容器间的关联，使用其他容器的IP、env等信息    
  --lxc-conf=[]              指定容器的配置文件，只有在指定--exec-driver=lxc时使用    
  --name=""                  指定容器名字，后续可以通过名字进行容器管理，links特性需要使用名字    
  --net="bridge"             容器网络设置:  
                                bridge 使用docker daemon指定的网桥       
                                host    //容器使用主机的网络    
                                container:NAME_or_ID  >//使用其他容器的网路，共享IP和PORT等网络资源    
                                none 容器使用自己的网络（类似--net=bridge），但是不进行配置   
  --privileged=false         指定容器是否为特权容器，特权容器拥有所有的capabilities    
  --restart="no"             指定容器停止后的重启策略:  
                                no：容器退出时不重启    
                                on-failure：容器故障退出（返回值非零）时重启   
                                always：容器退出时总是重启    
  --rm=false                 指定容器停止后自动删除容器(不支持以docker run -d启动的容器)    
  --sig-proxy=true           设置由代理接受并处理信号，但是SIGCHLD、SIGSTOP和SIGKILL不能被代理


docker install  refer: http://blog.sina.com.cn/s/blog_4d22b9720102x7v5.html
  
docker run -d --name zookeeper --publish 2181:2181 --volume /etc/localtime:/etc/localtime zookeeper:latest
	
docker run -d --name kafka --publish 9092:9092 --link zookeeper --env KAFKA_ZOOKEEPER_CONNECT=zookeeper:2181 --env KAFKA_ADVERTISED_HOST_NAME=localhost --env KAFKA_ADVERTISED_PORT=9092 --volume /etc/localtime:/etc/localtime wurstmeister/kafka:latest



删除docker镜像

docker rm -f ******

提交新镜像到私服务器

docker push docker.ops.colourlife.com:5000/nginx-php71-gs


更新私服务器镜像

sudo docker commit -m "添加ghostscript" -a "zhangxiwang" 5d99f3e442dd docker.ops.colourlife.com:5000/nginx-php71-gs

-m  描述   ，  -a 操作者
此时ID为5d99f3e442dd的容器， docker.ops.colourlife.com:5000/nginx-php71-gs为后来更新者



FROM docker.ops.colourlife.com:5000/nginx-php71-gs

RUN wget https://github.com/ArtifexSoftware/ghostpdl-downloads/releases/download/gs926/ghostscript-9.26-linux-x86_64.tgz \
&& tar -xzvf ghostscript-9.26-linux-x86_64.tgz  \
&& cd ghostscript-9.26-linux-x86_64 \
&& cp gs-926-linux-x86_64 /usr/bin/gs \
&& rm -rf ghostscript-9.26-linux-x86_64 ghostscript-9.26-linux-x86_64.tgz



RUN apt-get update \
&& apt-get install -y imagemagick

docker build -t docker.ops.colourlife.com:5000/nginx-php71-gs .

sudo docker commit -m "添加imagemagick" -a "zhangxiwang" 5d99f3e442dd docker.ops.colourlife.com:5000/nginx-php71-gs



我们运行的容器可能在镜像的基础上做了一些修改，有时候我们希望保存起来，封装成一个更新的镜像

docker自己提供的有commit功能

我们以centos为例，现在我们要在一个裸的centos上面安装vim编辑器，并且把这个功能保存下来，封装成一个能执行vim命令的centos镜像

拉去最新centos镜像：docker pull centos

进入镜像内部： docker run -it centos /bin/bash     备注：/bin/bash不要忘了

[root@202 ~]# docker run -it centos /bin/bash
[root@afcaf46e8305 /]#

afcaf46e8305是产生的容器ID，前面运行的时候不要-d后台运行了，不然会进不去容器内部的

[root@afcaf46e8305 /]# yum update

[root@afcaf46e8305 /]# yum install -y vim 

安装完了后：exit退出容器

然后把容器打包成镜像：

root@202 ~]# docker commit afcaf46e8305 centos-vim

完成后docker images查看镜像就会发现centos-vim这个镜像了

我们再用刚刚的方法进去centos-vim这个镜像,

[root@7f2d42f3e0a3 /]# vim --version

就可以看到vim的信息了

最后需要 docker push , 才能推到远程服务器，供下载使用

docker push docker.ops.colourlife.com:5000/nginx-php71-gs:latest

后面的 latest 表示是版本,要加