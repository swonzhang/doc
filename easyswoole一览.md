
最近自己也在使用swoole搭建框架，由于之前使用了easyswoole,自己在构思过程中，自然而然地会要借鉴easyswoole里面的一些精髓。但是随着工作进行下去，会感觉有许多的模糊的地方。所以现在做个整理一览出来。
_（注：本说明文档有点抽丝剥茧的意思，所以可能会对每个相关的内容进行有点深度的探究）_

==在开始之前，我还需得强调一句 ++一切皆对象++ ==

**我先列下几个提纲**

* 配置文件
* 服务命令 包括，启动服务，重启服务，关掉服务，主要探索启动服务前后的一系列操作（重点）
* 引入一系列composer组件，并在服务中（如何）使用，worker 生命周期
* 注册服务，路由，工具类
* 自定义进程、及自定义进程与worker 之间的通讯

- - -

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


- - -

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
```php
//EasySwoole\Core\Swoole\ServerManager 文件

//实例化对象池管理
PoolManager::getInstance();
$register->add($register::onWorkerStart,function (\swoole_server $server,int $workerId){
    PoolManager::getInstance()->__workerStartHook($workerId);
    $workerNum = Config::getInstance()->getConf('MAIN_SERVER.SETTING.worker_num');
    $name = Config::getInstance()->getConf('SERVER_NAME');
    if(PHP_OS != 'Darwin'){
        if($workerId <= ($workerNum -1)){
            $name = "{$name}_Worker_".$workerId;
        }else{
            $name = "{$name}_Task_Worker_".$workerId;
        }
        cli_set_process_title($name);
    }
});
EventHelper::registerDefaultOnTask($register);
EventHelper::registerDefaultOnFinish($register);
EventHelper::registerDefaultOnPipeMessage($register);
$conf = Config::getInstance()->getConf("MAIN_SERVER");
if($conf['SERVER_TYPE'] == self::TYPE_WEB_SERVER || $conf['SERVER_TYPE'] == self::TYPE_WEB_SOCKET_SERVER){
    if(!$register->get($register::onRequest)){
        EventHelper::registerDefaultOnRequest($register);
    }
}
```

`PoolManager::getInstance()`,这里是根据配置文件 **POOL_MANAGER**获取对象池配置，官方文档有提供了MYSQL连接池的例子，[文档链接](https://www.easyswoole.com/Manual/2.x/Cn/_book/CoroutinePool/mysql_pool.html)，当然这个需要在Config.php配置才会启用。==注意：easyswoole的连接池是保存在table里面的==，然后等到workerStart后，执行`PoolManager::getInstance()->__workerStartHook($workerId);`,在这个worker进程内生成连接池，那他这个连接池是怎么实现的呢，比如说你这个连接池要做五个对象，首先你就得生成五个对象，然后保存在table里，然后如何获取这个池呢，它提供了个`getPool`的方法，拿到连接池了，就可以通过`getObj`从连接池中取出一个对象，使用完之后，比如说是查完数据库后，就是用`freeObj`,释放这个对象，重新回到池里供使用。  

其实，`getObj`这里面也有文章，它是将连接池当做是一个 SplQueue队列对象，`getObj`就是出列，`freeObj`就是入列，倘若队列为空时，就是新建一个对象临时使用。具体的逻辑可以详见**EasySwoole\Core\Component\Pool\AbstractInterface\Pool**文件  

`cli_set_process_title($name);`接下来就是设置进程名称，使用`ps -ef|grep name`时会清晰点
继续就是，注册默认的回调事件了，easyswoole的回调事件都在`EasySwoole\Core\Swoole\EventHelper`文件里面定义好了，接下来我们就说说EventHelper里面的各个默认回调函数。  

`OnWorkerStart` 看上面代码，我们可以发现easyswoole只是做了对象池初始化处理，还有对worker进程的重命名处理。  

`EventHelper::registerDefaultOnTask($register);`  异步任务处理，easyswoole支持处理closure和class->method的时间回调，这也许就是他的风格，通过传输对象到执行处，再通过执行对象方法，来处理要执行的任务，要是通过后面一种方式处理，则需要继承抽象__EasySwoole\Core\Swoole\Task\AbstractAsyncTask__，其中有run()和finish()方法，分别执行启动任务和任务完成后的执行函数。值得一提到的是，使用了swoole的==barrier==,可执行多任务，以及对超时任务的处理。详细可参考官方文档 [点击查看](https://www.easyswoole.com/Manual/2.x/Cn/_book/Advanced/async_task.html)；  

`EventHelper::registerDefaultOnPipeMessage($register);`当工作进程收到由 [sendMessage](https://wiki.swoole.com/wiki/page/363.html) 发送的管道消息时会触发onPipeMessage事件。worker/task进程都可能会触发onPipeMessage事件。在easyswoole中，自定义进程发送异步任务，就是用了PipeMessage,推送消息到worker进程或task进程,具体可看**EasySwoole\Core\Swoole\Task\TaskManager::processAsync**,还有easyswoole是直接传输了一个序列化对象投递任务。  

这里需要注意一点，似乎目前现在的框架都喜欢使用闭包函数来处理回调，就相当于注册一个函数，然后在某个时间执行注册函数之时，再使用类似`call_user_func`的函数来执行闭包函数，这样的话，感觉是很优雅，也是主流，所以须得多多熟悉。  

`EventHelper::registerDefaultOnRequest($register);`在收到一个完整的Http请求后，会回调此函数。easyswoole在你个人没有设置==onRequest==,会自动注册此回调。这个默认回调主要是完成了对http请求的路由和异常的处理。上代码看看：
```php
   public static function registerDefaultOnRequest(EventRegister $register,$controllerNameSpace = 'App\\HttpController\\'):void
    {
        $dispatcher = new Dispatcher($controllerNameSpace);
        $register->set($register::onRequest,function (\swoole_http_request $request,\swoole_http_response $response)use($dispatcher){
            $request_psr = new Request($request);
            $response_psr = new Response($response);
            try{
                EasySwooleEvent::onRequest($request_psr,$response_psr);
                $dispatcher->dispatch($request_psr,$response_psr);
                EasySwooleEvent::afterAction($request_psr,$response_psr);
            }catch (\Throwable $throwable){
                $handler = Di::getInstance()->get(SysConst::HTTP_EXCEPTION_HANDLER);
                if($handler instanceof ExceptionHandlerInterface){
                    $handler->handle($throwable,$request_psr,$response_psr);
                }else{
                    $response_psr->withStatus(Status::CODE_INTERNAL_SERVER_ERROR);
                    $response_psr->write(nl2br($throwable->getMessage() ."\n". $throwable->getTraceAsString()));
                }
            }
            $response_psr->response();
        });
    }
```

对于其中的路由处理，我们先不深究，待下一个章节详细说明。我们可以看到，这里面包着一对`try{}catch`,我们前文提到easyswoole有四个全局事件，其中说道框架初始化时，就说到自定义捕捉异常，而这里就是处理自定义异常的地方。还有说到全局事件,这里就有两个：
```php
EasySwooleEvent::onRequest($request_psr,$response_psr); \\当EasySwoole收到任何的HTTP请求时，均会执行该事件。该事件可以对HTTP请求全局拦截。
$dispatcher->dispatch($request_psr,$response_psr); \\这里就是进行路由匹配处理，是我们处理业务逻辑的入口
EasySwooleEvent::afterAction($request_psr,$response_psr); \\请求方法结束后执行
```

至此，转了一大圈，其实也只是说了`ServerManager->finalHook()`这个函数内的注册资料而已。执行完该函数后，我们再回到`ServerManager->createMainServer()`,这时候执行到`EasySwooleEvent::mainServerCreate($this,$register)`这句了。这也是个全局事件，是在主服务启动前的最后自定义注册函数了，官方文档给我们指出，在此处，我们可以处理**注册主服务回调事件、添加一个自定义进程、添加一个子服务**。具体操作可看文档[点击查看](https://www.easyswoole.com/Manual/2.x/Cn/_book/Event/main_server_create.html)  

接下来就是对所有注册的事件，注入主服务对象，然后`createMainServer`函数执行完毕。  

继续回到`ServerManager->start`函数了,现在执行到  
`Cache::getInstance();`,这里是easyswoole的自定义缓存，也是使用table数据结构操作,这个我们下面说swoole组件时，会详细说。  

`Cluster::getInstance()->run();` 集群的执行，这个我们把它放到下文中详细去讲。  
`CronTab::getInstance()->run();`这个也是属于easyswoole的附带组件，下文详聊。  
`$this->attachListener();` swoole支持监听多服务，业务代码中可以通过调用swoole_server::connection_info来获取某个连接来自于哪个端口，主服务器是WebSocket或Http协议，新监听的TCP端口默认会继承主Server的协议设置。必须单独调用set方法设置新的协议才会启用新协议  
`$this->getServer()->start();`最后执行开启服务。  

至此，服务已完全启动。

- - -















