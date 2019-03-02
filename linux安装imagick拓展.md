首先,prerequisiteA后安装 php-pear
# yum install php-pear gcc
# yum install ImageMagick ImageMagick-devel ImageMagick-perl

接下来,编译imagick论坛 PHP 一个扩展
# pecl install imagick 

现在,添加一个 imagick.so 一个扩展到 /etc/php.ini 一个文件。
# echo extension=imagick.so >> /etc/php.ini

重启服务器
lnmp stop
lnmp start

通过运行下面的命令验证imagick PHP扩展。 下面你会看到imagick扩展相似。
# php -m | grep imagick
显示 imageick

或者通过phpinfo查看
--------------------- 
作者：非常cz伟 
来源：CSDN 
原文：https://blog.csdn.net/chenzhiwei666/article/details/78180249 
版权声明：本文为博主原创文章，转载请附上博文链接！