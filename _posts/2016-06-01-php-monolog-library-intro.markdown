---
layout: post
title: PHP 日志类库 monolog 介绍
category: php
comments: true
finished: true
---

>Monolog sends your logs to files, sockets, inboxes, databases and various web services.

Monolog 发送你的日志到文件、到sockets、到邮箱、到数据库或（和）者其他网路存储服务（云）。Monolog可以做到同时保存到一个或多个存储介质(后面的栈冒泡处理)。

## 安装

```bash
$ composer require monolog/monolog
```

## 基本用法 （初步印象）

```php
<?php
use Monolog\Logger;
use Monolog\Handler\StreamHandler;

// create a log channel
$log = new Logger('name');
$log->pushHandler(new StreamHandler('path/to/your.log', Logger::WARNING));

// add records to the log
$log->warning('Foo');$log->error('Bar');
```

## 核心概念 [¶官方解释](https://github.com/Seldaek/monolog/blob/master/doc/01-usage.md#core-concepts)

>Every `Logger` instance has a channel (name) and a stack of handlers. Whenever you add a record to the logger, it traverses the handler stack. Each handler decides whether it fully handled the record, and if so, the propagation of the record ends there.

每一个`Logger`实例都有一个**通道**（也就是一个唯一的名称）和一个有由一个或多个处理程序组成的**栈**。当我们添加一个记录到`Logger`的时候，它会遍历这个处理程序栈。每一个处理程序决定是否去充分处理这个记录，如果是，则处理到此为止（停止冒泡）。这里的**充分**指的是我们想不想了，想的话就继续，不想就停止。

这就允许我们灵活的设置日志了。比如我们有一个`StreamHandler`，它在**栈**的最底部，它会把记录都保存到硬盘上，在它上面有一个`MailHandler`，它会在错误消息被记录的时候发送邮件。`Handlers` 都有一个`$bubble`属性，用来定义当某个处理程序在处理记录的时候是否阻塞处理（阻塞的话，就是这个记录到我这里就算处理完毕了，不要冒泡处理了，听话）。在这个例子中，我们设置`MailHandler`的`$bubble`为`false`，意思就是说记录都会被`MailHandler`处理，不会冒泡到`StreamHandler`了。

>补充一下：这里提到了栈，也提到了冒泡，乍一看有点晕，因为我们理解冒泡是自下而上的过程，栈就是一个类似杯子的容器，然后上面又说底部是`StreamHandler`，上面是`MailHandler`，结果是`MailHandler`处理了，停止冒泡到`StreamHandler`了，给人的感觉是这个泡是从上往下冒的，666，这能叫冒泡么？**英雄时刻：**堆栈啥的，我也偶尔混淆，这里再次谨记，堆是先进先出（**F**irst-**I**n/**F**irst-**O**ut），想想[自来]水管；栈就是先进后出（**F**irst-**I**n/**L**ast-**O**ut）,想想一个有N层颜色的冰淇淋装在一个杯子里，下面是黄色的，...，最上面是粉红的，所以，你先吃得是粉红色的（`MailHandler`）,后吃的是黄色的（`StreamHandler`）,实际上，这个泡冒的没错，确切的说，这个泡冒在了一个倒立的杯子中，这样理解就形象了。

继续...

我们可以创建很多`Logger`,每个`Logger`定义一个通道（e.g.:db,request,router,...），每个通道可结合多个Handler，Handler可以被写成可通用的或者不可通用的。通道，同日志中日期时间一样，它是一个名称，在日志中就是一个字符串被记录下来，大概是这样 `2016-04-25 12:33:00 通道名称 记录内容`，具体格式看设置了，可以用来识别或者过滤。

每一个Handler都有一个`Formatter`，用来格式化日志了。不详细介绍了。

自定义日志等级在monolog中不可用，只有8种[RFC 5424](http://tools.ietf.org/html/rfc5424) 等级，即 debug, info, notice, warning, error, critical, alert, emergency。但是如果我们真的有特殊需求的话，比如归类等，我们可以添加`Processors`到 Logger，当然是在日志消息被处理之前。<strike>我这估计这辈子都不会添加`Processors`。</strike>

## 日志等级
* **DEBUG** (100): Detailed debug information.详细的Debug信息
* **INFO** (200): Interesting events. Examples: User logs in, SQL logs.感兴趣的事件或信息，如用户登录信息，SQL日志信息
* **NOTICE** (250): Normal but significant events.普通但重要的事件信息
* **WARNING** (300): Exceptional occurrences that are not errors. Examples: Use of deprecated APIs, poor use of an API, undesirable things that are not necessarily wrong.
* **ERROR** (400): Runtime errors that do not require immediate action but should typically be logged and monitored.
* **CRITICAL** (500): Critical conditions. Example: Application component unavailable, unexpected exception.
* **ALERT** (550): Action must be taken immediately. Example: Entire website down, database unavailable, etc. This should trigger the SMS alerts and wake you up.
* **EMERGENCY** (600): Emergency: system is unusable.

## 配置一个Logger
Here is a basic setup to log to a file and to firephp on the DEBUG level:

```php
<?php
use Monolog\Logger;
use Monolog\Handler\StreamHandler;
use Monolog\Handler\FirePHPHandler;

// Create the logger
$logger = new Logger('my_logger');

// Now add some handlers
$logger->pushHandler(new StreamHandler(__DIR__.'/my_app.log', Logger::DEBUG));
$logger->pushHandler(new FirePHPHandler());

// You can now use your logger
$logger->addInfo('My logger is now ready');
```

我们来分析一下这个配置。
The first step is to create the logger instance which will be used in your code. The argument is a channel name, which is useful when you use several loggers (see below for more details about it).
第一步，创建`Logger`实例，参数即通道名字。

The logger itself does not know how to handle a record. It delegates it to some handlers. The code above registers two handlers in the stack to allow handling records in two different ways.
`Logger`本身不知道如何处理记录，它将处理委托给`Handler`[s]，上面的代码注册了两个`Handler`s，这样就可以用两种方法来处理记录。

Note that the FirePHPHandler is called first as it is added on top of the stack. This allows you to temporarily add a logger with bubbling disabled if you want to override other configured loggers.
提示：`FirePHPHandler`最先被调用，因为它被添加在栈的顶部。这就允许你临时添加一个阻塞的`Logger`，如果你想覆盖其他`Logger`[s]的话。

## 添加额外的数据到记录
Monolog 提供两种方法来添加额外的信息到简单的文本信息（along the simple textual message）。

### 使用日志上下文

第一种，即当前日志上下文，允许传递一个数组作为第二个参数，这个数组的数据是额外的信息：

```php
<?php

$logger->addInfo('Adding a new user', array('username' => 'Seldaek'));
```

简单的Handler（SteamHandler）会简单的将数组格式化为字符串，功能丰富点的Handler（FirePHP）可以搞得更好看。

### 使用 processors

Processors 可以是任何可调用的方法（回调）。它们接受`$record`作为参数，然后返回它（`$record`）,返回之前，即是我们添加**额外信息**的操作，在这里，这个操作是改变`$record`的`extra`key的值。像这样：

```php
<?php

$logger->pushProcessor(function ($record) { 
    $record['extra']['dummy'] = 'Hello world!'; 
    return $record;
});
```

Monolog 提供了一些内置的 processors。看[dedicated chapter](https://github.com/Seldaek/monolog/blob/master/doc/02-handlers-formatters-processors.md#processors) 
收回我说的话，我可能很快就会用到 Processors的。

## 使用通道
通道是识别record记录的是程序哪部分的好方法（当然，关键词匹配啊），这在大型应用中很有用，如 MonologBundle in Symfony2。

想象一下，两个`Logger`共用一个`Handler`，通过这个`Handler`将记录写入一个文件。这时使用通道能够让我们识别出是哪一个`Logger`处理的。我们可简单的在这个文件中过滤这个或者那个通道。

```php
<?php

use Monolog\Logger;
use Monolog\Handler\StreamHandler;
use Monolog\Handler\FirePHPHandler;

// Create some handlers
$stream = new StreamHandler(__DIR__ . '/my_app.log', Logger::DEBUG);
$firephp = new FirePHPHandler();

// Create the main logger of the app
$logger = new Logger('my_logger');
$logger->pushHandler($stream);
$logger->pushHandler($firephp);

// Create a logger for the security-related stuff with a different channel
$securityLogger = new Logger('security');
$securityLogger->pushHandler($stream);
$securityLogger->pushHandler($firephp);

// Or clone the first one to only change the channel
$securityLogger = $logger->withName('security');
```

## 自定义日志格式
在 Monolog 中个性化日志是很easy的。大部分 Handler 使用`$record['formatted']`的值。这个值依赖于 formatter 的设置。我们可以选择预定义的 formatter 类或者编写自己的。

配置一个预定义的 formatter 类，只需要将其设置成 Handler 的字段（属性）即可：

```php
<?php
// the default format is "Y-m-d H:i:s"
$dateFormat = "Y n j, g:i a";
// the default output format is [%datetime%] %channel%.%level_name%: %message% %context% %extra%\n"
$output = "%datetime% > %level_name% > %message% %context% %extra%\n";
$formatter = new LineFormatter($output, $dateFormat);

// Create a handler
$stream = new StreamHandler(__DIR__ . 'my_app.log', Logger:DEBUG);
$stream->setFormatter($formatter);
// bind it to a logger object
$securityLogger = new Logger('security');
$securityLogger->pushHandler($stream);
```

formatter 是可以在N个 Handler 之间复用的，并且可在N个 Logger 之间共享 Handler。

## Handlers

### 记录日志到文件与系统日志（syslog）

* *StreamHandler*：记录日志到任何 PHP stream，用它来记录到文件。
* *RotatingFileHandler*: 每天一个文件，会自动删除比`$maxFiles`老的文件，这只是一个很随意的方案，You should use [logrotate](http://linuxcommand.org/man_pages/logrotate8.html) for high profile setups though。
* *SyslogHandler*: 记录到系统日志
* *ErrorLogHandler*:  Logs records to PHP's [error_log()
](http://docs.php.net/manual/en/function.error-log.php) function.

----

**扩展一下...**

----


### 发送提醒与邮件

>看看就能理解

* **NativeMailerHandler**: Sends emails using PHP's [mail()
](http://php.net/manual/en/function.mail.php) function.

* **SwiftMailerHandler**: Sends emails using a [Swift_Mailer
](http://swiftmailer.org/) instance.

* **PushoverHandler**: Sends mobile notifications via the [Pushover](https://www.pushover.net/) API.

* **HipChatHandler**: Logs records to a [HipChat](http://hipchat.com/) chat room using its API.

* **FlowdockHandler**: Logs records to a [Flowdock](https://www.flowdock.com/) account.

* **SlackHandler**: Logs records to a [Slack](https://www.slack.com/) account.

* **MandrillHandler**: Sends emails via the Mandrill API using a [Swift_Message
](http://swiftmailer.org/) instance.

* **FleepHookHandler**: Logs records to a [Fleep](https://fleep.io/) conversation using Webhooks.

* **IFTTTHandler**: Notifies an [IFTTT](https://ifttt.com/maker) trigger with the log channel, level name and message.

### 记录到指定server与网络日志

> 接着看看

* **SocketHandler**: Logs records to [sockets](http://php.net/fsockopen), use this for UNIX and TCP sockets. See an [example](https://github.com/Seldaek/monolog/blob/master/doc/sockets.md).

* **AmqpHandler**: Logs records to an [amqp](http://www.amqp.org/) compatible server. Requires the [php-amqp](http://pecl.php.net/package/amqp) extension (1.0+).

* **GelfHandler**: Logs records to a [Graylog2](http://www.graylog2.org/) server.

* **CubeHandler**: Logs records to a [Cube](http://square.github.com/cube/) server.

* **RavenHandler**: Logs records to a [Sentry](http://getsentry.com/) server using [raven](https://packagist.org/packages/raven/raven).

* **ZendMonitorHandler**: Logs records to the Zend Monitor present in Zend Server.

* **NewRelicHandler**: Logs records to a [NewRelic](http://newrelic.com/) application.

* **LogglyHandler**: Logs records to a [Loggly](http://www.loggly.com/) account.

* **RollbarHandler**: Logs records to a [Rollbar](https://rollbar.com/) account.

* **SyslogUdpHandler**: Logs records to a remote [Syslogd](http://www.rsyslog.com/) server.

* **LogEntriesHandler**: Logs records to a [LogEntries](http://logentries.com/) account.

### 开发环境中，利用浏览器扩展

> 装扩展

* **FirePHPHandler**: Handler for [FirePHP](http://www.firephp.org/), providing inline console
 messages within [FireBug](http://getfirebug.com/).

* **ChromePHPHandler**: Handler for [ChromePHP](http://www.chromephp.com/), providing inline console
 messages within Chrome.

* **BrowserConsoleHandler**: Handler to send logs to browser's Javascript console
 with no browser extension required. Most browsers supporting console
 API are supported.

* **PHPConsoleHandler**: Handler for [PHP Console](https://chrome.google.com/webstore/detail/php-console/nfhmhhlpfleoednkpnnnkolmclajemef), providing inline console
 and notification popup messages within Chrome.

### 记录到数据库

> 顾名思义

* **RedisHandler**: Logs records to a [redis](http://redis.io/) server.

* **MongoDBHandler**: Handler to write records in MongoDB via a [Mongo](http://pecl.php.net/package/mongo) extension connection.

* **CouchDBHandler**: Logs records to a CouchDB server.

* **DoctrineCouchDBHandler**: Logs records to a CouchDB server via the Doctrine CouchDB ODM.

* **ElasticSearchHandler**: Logs records to an Elastic Search server.

* **DynamoDbHandler**: Logs records to a DynamoDB table with the [AWS SDK](https://github.com/aws/aws-sdk-php).

### 特殊的Handler

> 慢慢看

* **FingersCrossedHandler**: A very interesting wrapper. It takes a logger as parameter and will accumulate log records of all levels until a record exceeds the defined severity level. At which point it delivers all records, including those of lower severity, to the handler it wraps. This means that until an error actually happens you will not see anything in your logs, but when it happens you will have the full information, including debug and info records. This provides you with all the information you need, but only when you need it.

* **DeduplicationHandler**: Useful if you are sending notifications or emails when critical errors occur. It takes a logger as parameter and will accumulate log records of all levels until the end of the request (or flush()
 is called). At that point it delivers all records to the handler it wraps, but only if the records are unique over a given time period (60seconds by default). If the records are duplicates they are simply discarded. The main use of this is in case of critical failure like if your database is unreachable for example all your requests will fail and that can result in a lot of notifications being sent. Adding this handler reduces the amount of notifications to a manageable level.

* **WhatFailureGroupHandler**: This handler extends the *GroupHandler* ignoring exceptions raised by each child handler. This allows you to ignore issues where a remote tcp connection may have died but you do not want your entire application to crash and may wish to continue to log to other handlers.

* **BufferHandler**: This handler will buffer all the log records it receives until close()
 is called at which point it will callhandleBatch()
 on the handler it wraps with all the log messages at once. This is very useful to send an email with all records at once for example instead of having one mail for every log record.

* **GroupHandler**: This handler groups other handlers. Every record received is sent to all the handlers it is configured with.

* **FilterHandler**: This handler only lets records of the given levels through to the wrapped handler.

* **SamplingHandler**: Wraps around another handler and lets you sample records if you only want to store some of them.

* **NullHandler**: Any record it can handle will be thrown away. This can be used to put on top of an existing handler stack to disable it temporarily.

* **PsrHandler**: Can be used to forward log records to an existing PSR-3 logger

* **TestHandler**: Used for testing, it records everything that is sent to it and has accessors to read out the information.

* **HandlerWrapper**: A simple handler wrapper you can inherit from to create your own wrappers easily.

## Formatters

✪ 为常用

* **LineFormatter**: Formats a log record into a one-line string. ✪

* **HtmlFormatter**: Used to format log records into a human readable html table, mainly suitable for emails.✪

* **NormalizerFormatter**: Normalizes objects/resources down to strings so a record can easily be serialized/encoded.

* **ScalarFormatter**: Used to format log records into an associative array of scalar values.

* **JsonFormatter**: Encodes a log record into json.✪

* **WildfireFormatter**: Used to format log records into the Wildfire/FirePHP protocol, only useful for the FirePHPHandler.

* **ChromePHPFormatter**: Used to format log records into the ChromePHP format, only useful for the ChromePHPHandler.

* **GelfMessageFormatter**: Used to format log records into Gelf message instances, only useful for the GelfHandler.

* **LogstashFormatter**: Used to format log records into [logstash](http://logstash.net/) event json, useful for any handler listed under inputs [here](http://logstash.net/docs/latest).

* **ElasticaFormatter**: Used to format log records into an Elastica\Document object, only useful for the ElasticSearchHandler.

* **LogglyFormatter**: Used to format log records into Loggly messages, only useful for the LogglyHandler.

* **FlowdockFormatter**: Used to format log records into Flowdock messages, only useful for the FlowdockHandler.

* **MongoDBFormatter**: Converts \DateTime instances to \MongoDate and objects recursively to arrays, only useful with the MongoDBHandler.

## Processors

* **PsrLogMessageProcessor**: Processes a log record's message according to PSR-3 rules, replacing {foo}
 with the value from $context['foo'].

* **IntrospectionProcessor**: Adds the line/file/class/method from which the log call originated.

* **WebProcessor**: Adds the current request URI, request method and client IP to a log record.

* **MemoryUsageProcessor**: Adds the current memory usage to a log record.

* **MemoryPeakUsageProcessor**: Adds the peak memory usage to a log record.

* **ProcessIdProcessor**: Adds the process id to a log record.

* **UidProcessor**: Adds a unique identifier to a log record.

* **GitProcessor**: Adds the current git branch and commit to a log record.

* **TagProcessor**: Adds an array of predefined tags to a log record.



## Utilities

* **Registry**: The `Monolog\Registry`
 class lets you configure global loggers that you can then statically access from anywhere. It is not really a best practice but can help in some older codebases or for ease of use.
`Monolog\Registry`允许我们配置全局的`logger`，并且我们可以全局静态访问，这虽然不是最佳实践，但可以在某些老的代码库中提供一些帮助或者仅仅只是简单使用。

* **ErrorHandler**: The `Monolog\ErrorHandler`
 class allows you to easily register a Logger instance as an exception handler, error handler or fatal error handler.
`Monolog\ErrorHandler` 允许我们注册一个`Logger`实例作为一个异常处理句柄，错误处理句柄或者致命错误处理句柄。

* **ErrorLevelActivationStrategy**: Activates a FingersCrossedHandler when a certain log level is reached.
当达到某个日志等级的时候激活 `FingersCrossedHandler`。

* **ChannelLevelActivationStrategy**: Activates a FingersCrossedHandler when a certain log level is reached, depending on which channel received the log record.
当达到某个日志等级的时候激活 `FingersCrossedHandler`，取决于哪个通道收到日志信息。



## Extending Monolog

Monolog 是完全可以扩展的。可以轻松让logger适用于我们的需求。

### 编写自己的 Handler

虽然 Monolog 提供了很多内置的 Handler，但是我们依然可能没有找到我们想要的那个，这时我们就要来编写并使用自己的了。仅需 **implement** `Monolog\Handler\HandlerInterface`。

来写个 PDOHandler，用来把日志存到数据库，我们继承 Monolog 提供的抽象类，以坚守 **Don't Rpeat Yourself** 原则。

```php
    <?php

    use Monolog\Logger;
    use Monolog\Handler\AbstractProcessingHandler;

    class PDOHandler extends AbstractProcessingHandler
    {
        private $initialized = false;
        private $pdo;
        private $statement;

        public function __construct(PDO $pdo, $level = Logger::DEBUG, $bubble = false)
        {
            $this->pdo = $pdo;
            parent::__construct($level, $bubble);
        }
        
        protected function write(array $record)
        {   
             if (!$this->initialized) {
                 $this->initialize();
             }

           $this->statement->execute(array(
                'channel' => $record['channel'],
                'level' => $record['level'],
                'message' => $record['formatted'],
                'time' => $record['datetime']->format('U'),             
           ));
        }
        
        private function initialize()
        {
            $this->pdo->exec(
                'CREATE TABLE IF NOT EXISTS monolog ' .'(channel VARCHAR(255), level INTEGER, message LONGTEXT, time INTEGER UNSIGNED)'
            );
            $this->statement = $this->pdo->prepare(
                'INSERT INTO monolog (channel, level, message, time) VALUES (:channel, :level, :message, :time)'
            );

            $this->initialized = true;
        }
    }
```

现在就可以在Logger中使用这个Handler了：

```php

$logger->pushHandler(new PDOHandler(new PDO('sqlite:logs.sqlite')));

// You can now use your logger
$logger->addInfo('My logger is now ready');
```

`Monolog\Handler\AbstractProcessingHandler`提供了Handler需要的大部分逻辑，包括processors的使用以及record的格式化（which is why we use `$record['formatted']`
 instead of `$record['message']`）。