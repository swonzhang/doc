
最近自己也在使用swoole搭建框架，由于之前使用了easyswoole,自己在构思过程中，自然而然地会要借鉴easyswoole里面的一些精髓。但是随着工作进行下去，会感觉有许多的模糊的地方。所以现在做个整理一览出来。
_（注：本说明文档有点抽丝剥茧的意思，所以可能会对每个相关的内容进行有点深度的探究）_

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
>了解了下php的引用特性，_划重点_，**如果对一个未定义的变量进行引用赋值、引 用参数传递或引用返回，则会自动创建该变量**。[点击查看参考文档](http://php.net/manual/zh/language.references.whatdo.php)


### 服务命令

即为 **php  easyswoole start|restart|reload|stop|install** 几种命令
其内涵都在 easywoole 文件里面里，文件位置一般在 _vender/bin/easyswoole_

**下面开始分别说下各种命令：**

```shell
php easyswoole start
```
让我们探索在命令行下执行这行代码后，easyswoole到底做了什么？ 看代码[^1]

```php
// easyswoole 文件
case 'start':
    installCheck();  // 检查easyswoole框架是否搭建了起来，这个函数其实是判断easyswoole.install 文件是否存在来判断
    serverStart($options); //这才是主心骨
```
_serverStart()_ 这个函数才是主要执行内容。而在该函数中最重要的又是以下：
```php
//easyswoole 文件
$conf    = Conf::getInstance();  // 这个就是获取配置文件config.php的信息，如何获取的可看前文的配置文件章节
$inst    = Core::getInstance()->initialize(); // 一目了然，这就获取核心函数的实例，执行初始化函数，所以这是**重点之处**哦， use EasySwoole\Core\Core;
```
接下来我们看重点之处的文件，来于 **use EasySwoole\Core\Core;** 首先让我们看下 **initialize（）**函数。
```php
//EasySwoole\Core\Core 文件
Di::getInstance()->set(SysConst::VERSION,'2.1.1');
Di::getInstance()->set(SysConst::HTTP_CONTROLLER_MAX_DEPTH,3);
//创建全局事件容器
$event = $this->eventHook();
$this->sysDirectoryInit();
$event->hook('frameInitialize');
$this->errorHandle();
return $this;
```
*`Di::getInstance()->set(SysConst::VERSION,'2.1.1');`这个对核心内容无大牵涉，但我还是向谈谈，就是配置文件内容，是处于对象四层生命周期(程序全局期，进程全局期、会话期、请求期)的程序全局期，也就是说一旦程序启动，是没法通过reload来重新加载配置文件的，但是却可以在服务启动前，通过代码修改配置或者设置配置信息*

接下来我的重点介绍**全局事件容器**这个东东，`$event = $this->eventHook();`,这语句就是`EasySwooleEvent`前要准备，easyswoole的有四个全局事件（框架初始化事件、主服务创建事件、请求事件、响应后事件），而`$event->hook('frameInitialize');`就是执行框架初始化处理，按照easyswoole文档的说明，这里框架初始化，可以做全局异常捕捉处理（$this->errorHandle();）、创建临时目录(`$this->sysDirectoryInit();`)、还有其它的设置配置文件的操作;
`$this->sysDirectoryInit()`，主要是设置程序pid文件，和swoole.log文件
`$this->errorHandle()`,主要是设置异常注册和处理，对于异常处理可以参考 [PHP错误与异常处理](https://www.cnblogs.com/zyf-zhaoyafei/p/6928149.html)
再说多一句，在执行初始化框架时，easyswoole提供了，自定义的捕捉异常处理，` Di::getInstance()->set( SysConst::HTTP_EXCEPTION_HANDLER, \App\ExceptionHandler::class );`,可以自定义抛出异常，这是在==onRequest==中处理的，后面会提到。

到此，`use EasySwoole\Core\Core` 已执行完毕，继续回到easyswoole文件
```php
// easyswoole 文件
$inst = Core::getInstance()->initialize();
//输出其他的展示数据
//
$inst->run();  // 至此，开始进入swoole的世界
```
执行run 之后，其实是执行 ==core==里面的`ServerManager::getInstance()->start();`,好戏开始了。让我们看看==start==干了啥。
```php
//EasySwoole\Core\Swoole\ServerManager 文件
public function start():void
    {
        $this->createMainServer(); //这这里就是完成主服务启动之前的配置，回调函数、以及其他的前要处理。
        Cache::getInstance();  // swoole提供的内存操作模块，使用的是 table
        Cluster::getInstance()->run(); //开始执行集群配置
        CronTab::getInstance()->run(); //开始执行定义任务
        $this->attachListener(); // swoole支持多个监听任务
        $this->isStart = true; // 确认已经执行任务
        $this->getServer()->start(); // 开始执行swoole模块
    }
```
这里先说`$this->createMainServer()`,看代码
```php
//EasySwoole\Core\Swoole\ServerManager 文件

$this->mainServer->set($setting); // 设置服务配置
//创建默认的事件注册器
$register = new EventRegister(); // 即是设置onWorkerStart等等事件
$this->finalHook($register); //
EasySwooleEvent::mainServerCreate($this,$register);
$events = $register->all();
foreach ($events as $event => $callback){
    $this->mainServer->on($event, function () use ($callback) {
        $ret = [];
        $args = func_get_args();
        foreach ($callback as $item) {
            array_push($ret,Invoker::callUserFuncArray($item, $args));
        }
        if(count($ret) > 1){
            return $ret;
        }
        return array_shift($ret);
    });
}
return $this->mainServer;

```
`$this->finalHook($register)`,这个单独拿出来讲一讲，做了*实例化对象池管理*,还有绑定各种回调函数。













