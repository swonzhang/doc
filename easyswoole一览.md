
最近自己也在使用swoole搭建框架，由于之前使用了easyswoole,自己在构思过程中，自然而然地会要借鉴easyswoole里面的一些精髓。但是随着工作进行下去，会感觉有许多的模糊的地方。所以现在做个整理一览出来。

**我先列下几个提纲**

* 配置文件
* 服务命令 包括，启动服务，重启服务，关掉服务，主要探索启动服务前后的一系列操作（重点）
* 引入一系列composer组件，并在服务中（如何）使用，worker 生命周期
* 请求路由
* 注册服务，
* 自定义进程、及自定义进程与worker 之间的通讯

### 配置文件

easyswoole 用的php 数组配置格式，当然是简单方便，直接用 splArray 继承 ArrayObject (spl接口)，ArrayObject很强大，但以我的水平，只能看出splArray只拿ArrayObject作为一个基本的数组对象载体。最主要的无非是set 和 get。splArray其亮点应该是支持点分式获取配置数据，利用了对象的特性，(关键函数 array_shift、temp= &temp[key]);

**例如：**


```php
<?php
    $obj = new stdclass();
    $temp = &$obj['gg4']['vvv'];  // 最神奇的就是可以直接赋予更深层数值
    $temp = 'ffff';
    $copy = $obj->getArrayCopy();
    echo "<pre>";
    print_r($copy);
```
		输出:
```php
Array(
	[gg4] => Array
        (
            [vvv] => ffff
        )
 )
```
>了解了下php的引用特性，_划重点_，**如果对一个未定义的变量进行引用赋值、引 用参数传递或引用返回，则会自动创建该变量**。


### 服务命令

即为 **php  easyswoole start|restart|reload|stop|install** 几种模式
其内涵都在 easywoole 文件里面里，文件位置一般在 _vender/bin/easyswoole_

**下面开始分别说下各种命令：**

```shell
php easyswoole start
```
