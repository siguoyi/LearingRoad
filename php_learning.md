# PHP 简介 #
## 什么是 PHP？ ##
PHP（“PHP: Hypertext Preprocessor”，超文本预处理器的字母缩写）是一种被广泛应用的开放源代码的多用途脚本语言，它可嵌入到 HTML中，尤其适合 web 开发。

- PHP 是 "PHP Hypertext Preprocessor" 的首字母缩略词
- PHP 是一种被广泛使用的开源脚本语言
- PHP 脚本在服务器上执行
- PHP 没有成本，可供免费下载和使用

以上是一个简单的回答，不过这是什么意思呢？请看如下例子：

    <html>
    <head>
        <title>Example</title>
    </head>
    <body>

        <?php
        echo "Hi, I'm a PHP script!";
        ?>

    </body>
	</html>
PHP 代码被包含在特殊的起始符和结束符 **<?php 和 ?>** 中，使得可以进出“PHP 模式”。使用 PHP 的一大好处是它对于初学者来说极其简单，同时也给专业的程序员提供了各种高级的特性。当看到 PHP 长长的特性列表时，请不要害怕。可以很快的入门，只需几个小时就可以自己写一些简单的脚本。
## 什么是 PHP 文件？ ##
- PHP 文件能够包含文本、HTML、CSS 以及 PHP 代码
- PHP 代码在服务器上执行，而结果以纯文本返回浏览器
- PHP 文件的后缀是 ".php"
## PHP 能够做什么？ ##
PHP 能做任何事。PHP 主要是用于服务端的脚本程序，因此可以用 PHP 来完成任何其它的 CGI 程序能够完成的工作，例如收集表单数据，生成动态网页，或者发送／接收 Cookies。但 PHP 的功能远不局限于此。

- PHP 能够生成动态页面内容
- PHP 能够创建、打开、读取、写入、删除以及关闭服务器上的文件
- PHP 能够接收表单数据
- PHP 能够发送并取回 cookies
- PHP 能够添加、删除、修改数据库中的数据
- PHP 能够限制用户访问网站中的某些页面
- PHP 能够对数据进行加密
## 为什么使用 PHP？ ##
- PHP 运行于各种平台（Windows, Linux, Unix, Mac OS X 等等）
- PHP 兼容几乎所有服务器（Apache, IIS 等等）
- PHP 支持多种数据库
- PHP 是免费的。请从官方 PHP 资源下载：[www.php.net](www.php.net)
- PHP 易于学习，并可高效地运行在服务器端
## PHP 应用领域 ##
PHP 脚本主要用于以下三个领域：

- **服务端脚本。**这是 PHP 最传统，也是最主要的目标领域。开展这项工作需要具备以下三点：PHP 解析器（CGI 或者服务器模块）、web 服务器和 web 浏览器。需要在运行 web 服务器时，安装并配置 PHP，然后，可以用 web 浏览器来访问 PHP 程序的输出，即浏览服务端的 PHP 页面。如果只是实验 PHP 编程，所有的这些都可以运行在自己家里的电脑中。请查阅[安装](http://php.net/manual/zh/install.php)一章以获取更多信息。
- **命令行脚本。**可以编写一段 PHP 脚本，并且不需要任何服务器或者浏览器来运行它。通过这种方式，仅仅只需要 PHP 解析器来执行。这种用法对于依赖 cron（Unix 或者 Linux 环境）或者 Task Scheduler（Windows 环境）的日常运行的脚本来说是理想的选择。这些脚本也可以用来处理简单的文本。请参阅 [PHP 的命令行模式](http://php.net/manual/zh/features.commandline.php)以获取更多信息。
- **编写桌面应用程序。**对于有着图形界面的桌面应用程序来说，PHP 或许不是一种最好的语言，但是如果用户非常精通 PHP，并且希望在客户端应用程序中使用 PHP 的一些高级特性，可以利用 PHP-GTK 来编写这些程序。用这种方法，还可以编写跨平台的应用程序。PHP-GTK 是 PHP 的一个扩展，在通常发布的 PHP 包中并不包含它。