# Beanbun
Beanbun 是用 PHP 编写的多进程网络爬虫框架，具有良好的开放性、高可扩展性。

# 安装

Beanbun 可以通过 composer 进行安装。

```
$ composer require kiddyu/beanbun
```

# 快速开始

创建一个文件 start.php，包含以下内容

```
<?php
require_once(__DIR__ . '/vendor/autoload.php');

use Beanbun\Beanbun;
$beanbun = new Beanbun;
$beanbun->seed = [
    'http://www.950d.com/',
    'http://www.950d.com/list-1.html',
    'http://www.950d.com/list-2.html',
];
$beanbun->afterDownloadPage = function($beanbun) {
    file_put_contents(__DIR__ . '/' . md5($beanbun->url), $beanbun->page);
};
$beanbun->start();
```

在命令行中执行

```
$ php start.php
```

接下来就可以看到抓取的日志了。

# 使用

启动与停止

上面的例子中，爬虫是以普通模式运行的，上面的代码放在网站项目中，也可以正常执行，如果我们想让爬虫一直执行，就需要使用守护进程模式。同样是上面的代码，我们只需要把执行的命令增加一个 start 参数，即会变成守护进程模式。

`$ php start.php start`

需要说明的是，普通模式下不依赖队列，爬虫只爬取 seed 中得地址，依次爬取完成后，程序即结束。而守护进程模式需要另外开启队列（内存队列、Redis 队列等），但拥有更多的功能，如可以自动发现页面中的链接加入队列，循环爬取。以下是守护进程模式下的说明。

启动

// 启动爬虫，开启所有爬虫进程
`$ php start.php start`

停止

// 停止爬虫，关闭所有爬虫进程
`php start.php stop`

清理

// 删除日志文件，清空队列信息
`php start.php clean`

在守护模式中，如果需要使用数据库、redis 等连接，需要在各种回调函数中建立连接，否则可能会发生意想不到的错误。
建议使用单例模式，并在 startWorker 中关闭之前建立的连接。

# 例子

例子一

爬取糗事百科热门列表页，采用守护进程模式。在开始爬取前，我们需要一个队列，在这里使用框架中带有的内存队列。
首先建立一个队列文件 queue.php，写入下列内容

```
<?php
require_once(__DIR__ . '/vendor/autoload.php');
// 启动队列
\Beanbun\Queue\MemoryQueue::server();
```
建立爬虫文件 start.php，写入下列内容

```
<?php
use Beanbun\Beanbun;
use Beanbun\Lib\Helper;

require_once(__DIR__ . '/vendor/autoload.php');

$beanbun = new Beanbun;
$beanbun->name = 'qiubai';
$beanbun->count = 5;
$beanbun->seed = 'http://www.qiushibaike.com/';
$beanbun->max = 30;
$beanbun->logFile = __DIR__ . '/qiubai_access.log';
$beanbun->urlFilter = [
    '/http:\/\/www.qiushibaike.com\/8hr\/page\/(\d*)\?s=(\d*)/'
];
// 设置队列
$beanbun->setQueue('memory', [
    'host' => '127.0.0.1',
     'port' => '2207'
 ]);
$beanbun->afterDownloadPage = function($beanbun) {
    file_put_contents(__DIR__ . '/' . md5($beanbun->url), $beanbun->page);
};
$beanbun->start();
```

接下来在命令行中执行

```
$ php queue.php start
$ php start.php start
```

先启动队列进程，再启动爬虫。
