
###yum 安装文件是出现 xxx公匙尚未安装

yum update -y -nogpgcheck  或者 yum install packages_name -y --nogpgcheck 忘记哪一个生效了

不行的话，就更换国内源，妥妥的 refer: https://www.cnblogs.com/xjh713/p/7458437.html
 备份本地yum源
 mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo_bak 

 获取阿里yum源配置文件
 wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo 

 更新cache
 yum makecache 

 查看
 yum -y update 
 
###防火墙，对于服务，肯定要打开端口让人访问，不过我在虚拟机操作，一般都是直接关闭防火墙
   
  CentOs7 用的是firewall和 systemctl 管理
   关闭防火墙: systemctl disable firewalld
   开启某一个端口：firewall-cmd --zone=public --add-port=443/tcp --permanent
   重载防火墙：firewall-cmd --reload
  CentOs6 用的是iptable 和 service管理
   关闭防火墙：service iptable off
   开启某个端口：-A INPUT -m state --state NEW -m tcp -p tcp --dport 3306 -j ACCEPT

   注：要是都在局域网内，都却访问不了另外一台主机，可以 systemctl status firewalld 看看防火墙的状态是否开启

###软件安装
	1、apache 我直接用yum 源安装了 yum -y -q install apr-devel apr-util-devel openssl-devel pcre-devel mod_ssl expat-devel
	2、php  编译安装 可以从官网下载包，http://am1.php.net/distributions/php-7.2.4.tar.gz 

	下面安装步骤，这个很全步骤：https://blog.csdn.net/u010861514/article/details/51926575
	首先安装依赖：
	yum -y install gcc gcc-c++ libxml2 libxml2-devel bzip2 bzip2-devel libmcrypt libmcrypt-devel openssl openssl-devel libcurl-devel libjpeg-devel libpng-devel freetype-devel readline readline-devel libxslt-devel perl perl-devel psmisc.x86_64 recode recode-devel libtidy libtidy-devel

	我用的是 fastcgi php-fpm模式: 编译参数是 
	./configure --prefix=/usr/local/php7 --sysconfdir=/etc/php7 --with-config-file-path=/etc/php7 --enable-fpm --with-mysqli=mysqlnd --with-pdo-mysql=mysqlnd --with-mhash --with-openssl --with-zlib --with-bz2 --with-curl --with-libxml-dir --with-gd --with-jpeg-dir --with-png-dir --with-zlib --enable-mbstring --with-mcrypt --enable-sockets --with-iconv-dir --with-xsl --enable-zip --with-pcre-dir --with-pear --enable-session  --enable-gd-native-ttf --enable-xml --with-freetype-dir --enable-gd-jis-conv --enable-inline-optimization --enable-shared --enable-bcmath --enable-sysvmsg --enable-sysvsem --enable-sysvshm --enable-mbregex --enable-pcntl --with-xmlrpc --with-gettext --enable-exif --with-readline --with-recode --with-tidy --with-apxs2=/usr/bin/apxs
	
	注：由于我可能也想要用apache模块模式，所以我加上 --with-apxs2=/usr/bin/apxs    留意apxs的位置，是 httpd-devel包里的
	
	编译期间出了问题：configure: error: jpeglib.h not found. configure: error: png.h not found. 这个应该是GD库的问题
	解决办法：yum -y install libjpeg-devel libpng-devel
	
	继续出问题 ：configure: error: freetype-config not found.  解决办法：yum install freetype-devel
	继续出问题：configure: error: Please reinstall readline - I cannot find readline.h    解决办法：yum -y install readline-devel
	
	至此。终于编译完成，这时你有没有发现上面的一个规律，编译出了问题都是 直接 install ***-devel 包就可以解决了，我去查了资料发现 *-devel是一类开发包，主要包括了一些头文件和静态链接。在某些模块编译时，需要依赖这些 *-devel包。当然，有些包直接用yum install会出现 “没有可用软件包 *-devel”的问题，那就得下载源码安装了.
	例如上面的 libtidy, 
	wget http://tidy.sourceforge.net/src/old/tidy_src_051026.tgz
	gunzip tidy-xxxx.tgz
	tar -xvf tidy-xxxx.tar
	cd tidy
	没有configure文件的话，可以使用sh build/gnuauto/setup.sh
	./configure --prefix=/usr/local/tidy
	make && make install
	编译到PHP，--with-tidy=/usr/local/tidy
	
	最后 make && make install 完成安装，其实，这又是个开始，直接启动php-fpm, /usr/local/php7/sbin/php-fpm 会报 [10-Apr-2018 10:55:57] ERROR: failed to open configuration file '/etc/php7/php-fpm.conf': No such file or directory (2)， 配置文件问题，需要把 /etc/php7目录下的default文件，复制一份没有.default后缀的即可，注意还有php-fpm.d目录下的www.conf.default也要复制一份 www.conf，这样，/usr/local/php7/sbin/php-fpm 就可以启动了 netstat -ano|grep 9000,php-fpm用的是9000端口。
	
	还有，这时你会发现，php.ini文件怎么没找着？ 我也郁闷了一会，然后就查找：find / -name php.ini* ,发现只有在源文件里有 php.ini.development文件，原来这个需要手动安装，只好复制一份 cp /usr/local/src/php-7.2.4/php.ini-development /etc/php7/php.ini
	
	php-fpm配置好了，该配置httpd了，打开httpd.conf文件
	
	首先检查 httpd模块是否打开
	 LoadModule proxy_module modules/mod_proxy.so  
         LoadModule proxy_fcgi_module modules/mod_proxy_fcgi.so
	上面两个是使支持代理？？有兴趣的可以自行资料查询

	找到 dir_module模块内，修改：DirectoryIndex index.html index.php
	找到 mime_module 模块内， 添加：AddType application/x-httpd-php .php 
	然后添加文件匹配模块：
	<FilesMatch \.php$>  
                             SetHandler proxy:fcgi://127.0.0.1:9000  
        </FilesMatch>
 	
	好，然后重启 httpd ； systemctl restart httpd; httpd和php安装完成 ！！！撒花。。。

###PHP拓展安装,PHP的拓展安装好像有两种方式, 一个是编译安装，一个是pecl安装下面分别说明
	1、编译安装，这个得下载源文件， 例如redis， 
		wget http://pecl.php.net/get/redis-4.0.0.tgz
		tar xzvf redis-4.0.0.tgz
		cd 	redis-4.0.0.tgz	
		/usr/local/php7/bin/phpize
		./configure --with-php-config=/usr/local/php/bin/php-config
		make && make install
		然后在 php.ini 文件里添加 extension = redis.so  重启 php-fpm，至此，php -m 就可以看到redis模块了
	2、通过pecl安装， 例如安装swoole模块,当然这个只能安装pecl收录的拓展
		直接 /usr/local/php7/bin/pecl install swoole
		然后在 php.ini 文件里添加 extension = redis.so  重启 php-fpm，至此，php -m 就可以看到redis模块了（不重启也可以看到，但实际上还没加载在php-fpm里）

	
	有些系统是没有pecl 的，需要自己安装 参考 https://blog.csdn.net/koastal/article/details/52850416

###用PHP怎么少得了composer呢，composer的安装也简单，直接下载 ： 

	wget https://getcomposer.org/download/1.6.3/composer.phar
	ln -s composer.phar /usr/local/bin/composer

	然后直接可以使用composer了，不过我觉得直接把composer.phar文件下载在/usr/local/bin文件直接使用更加方便，呵呵。。。
	噢，对了，在国内一般推荐使用中国镜像，感谢免费的镜像服务
	直接修改： composer config -g repo.packagist composer https://packagist.phpcomposer.com   

	至于如何使用composer,新建个 composer.json文件，然后 composer require "swonzhang/swon"  这样就可以把swon文件加载到vender文件里了

###编辑器，在window环境下，我习惯用sublime ,可是在linux系统里，sublime并不支持中文输入，好像ubantu可以使用插件解决；而centos则没找到太有效的资料。by the way, 解决Sublime包管理package control 报错 There are no packages available for installation, 参考：https://blog.csdn.net/qiangweiyan/article/details/79117641 ， schema_version 只能为 2.0或低版本，还有可能ip问题



###centos7下安装chrome浏览器，参考:https://blog.csdn.net/qq_36068954/article/details/80687971
	注意:不能翻墙的,gpgcheck=0 ,这个值设置为0；然后root不能启动，把SELINUX=disabled

	还会出现From what I can tell, the symbol gdk_screen_get_monitor_scale_factor 这个错误，只需要 ： yum install gtk3-devel

倘若安装失败，有可能 sudo yum remove libvirt-client
然后再安装： sudo yum install libreoffice-math

即可启动 chrome