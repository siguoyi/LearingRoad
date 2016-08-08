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

# PHP 语法 #
## 基本语法 ##
### PHP 标记 ###
当解析一个文件时，PHP 会寻找起始和结束标记，也就是 <?php 和 ?>，这告诉 PHP 开始和停止解析二者之间的代码。此种解析方式使得 PHP 可以被嵌入到各种不同的文档中去，而任何起始和结束标记之外的部分都会被 PHP 解析器忽略。

如果文件内容是纯 PHP 代码，最好在文件末尾删除 PHP 结束标记。这可以避免在 PHP 结束标记之后万一意外加入了空格或者换行符，会导致 PHP 开始输出这些空白，而脚本中此时并无输出的意图。

    <?php
	echo "Hello world";
	
	// ... more code
	
	echo "Last statement";
	
	// 脚本至此结束，并无 PHP 结束标记
### 从 HTML 中分离 ###
凡是在一对开始和结束标记之外的内容都会被 PHP 解析器忽略，这使得 PHP 文件可以具备混合内容。 可以使 PHP 嵌入到 HTML 文档中去，如下例所示。

    <p>This is going to be ignored by PHP and displayed by the browser.</p>
	<?php echo 'While this is going to be parsed.'; ?>
	<p>This will also be ignored by PHP and displayed by the browser.</p>
### 指令分隔符 ###
同 C 或 Perl 一样，PHP 需要在每个语句后用分号结束指令。一段 PHP 代码中的结束标记隐含表示了一个分号；在一个 PHP 代码段中的最后一行可以不用分号结束。如果后面还有新行，则代码段的结束标记包含了行结束。

    <?php
    echo "This is a test";
	?>
	
	<?php echo "This is a test" ?>
	
	<?php echo 'We omitted the last closing tag';
### 注释 ###
PHP 支持 C，C++ 和 Unix Shell 风格（Perl 风格）的注释。例如:

    <?php
    echo "This is a test"; // This is a one-line c++ style comment
    /* This is a multi line comment
       yet another line of comment */
    echo "This is yet another test";
    echo 'One Final Test'; # This is a one-line shell-style comment
	?>
## 类型 ##
### Boolean 布尔类型 ###
这是最简单的类型。boolean 表达了真值，可以为 TRUE 或 FALSE。要指定一个布尔值，使用关键字 TRUE 或 FALSE。两个都不区分大小写。

    <?php
		$foo = True; // assign the value TRUE to $foo
	?>
### Integer 整型 ###
一个 integer 是集合 ℤ = {..., -2, -1, 0, 1, 2, ...} 中的一个数。整型值可以使用十进制，十六进制，八进制或二进制表示，前面可以加上可选的符号（- 或者 +）。**二进制表达的 integer 自 PHP 5.4.0 起可用。**要使用八进制表达，数字前必须加上 0（零）。要使用十六进制表达，数字前必须加上 0x。要使用二进制表达，数字前必须加上 0b。

    <?php
	$a = 1234; // 十进制数
	$a = -123; // 负数
	$a = 0123; // 八进制数 (等于十进制 83)
	$a = 0x1A; // 十六进制数 (等于十进制 26)
	?>
### Float 浮点型  ###
浮点型（也叫浮点数 float，双精度数 double 或实数 real）可以用以下任一语法定义：

    <?php
	$a = 1.234; 
	$b = 1.2e3; 
	$c = 7E-10;
	?>
### String 字符串 ###
一个字符串 string 就是由一系列的字符组成，其中每个字符等同于一个字节。这意味着 PHP 只能支持 256 的字符集，因此不支持 Unicode 。一个字符串可以用 4 种方式表达：单引号，双引号，heredoc语法结构和nowdoc语法结构。
#### 单引号 ####
定义一个字符串的最简单的方法是用单引号把它包围起来（字符 '）。要表达一个单引号自身，需在它的前面加个反斜线（\）来转义。要表达一个反斜线自身，则用两个反斜线（\\）。其它任何方式的反斜线都会被当成反斜线本身：也就是说如果想使用其它转义序列例如 \r 或者 \n，并不代表任何特殊含义，就单纯是这两个字符本身。
#### 双引号 ####
如果字符串是包围在双引号（"）中。
#### Heredoc 结构 ####
第三种表达字符串的方法是用 heredoc 句法结构：<<<。在该运算符之后要提供一个标识符，然后换行。接下来是字符串 string 本身，最后要用前面定义的标识符作为结束标志。

结束时所引用的标识符必须在该行的第一列，而且，标识符的命名也要像其它标签一样遵守 PHP 的规则：只能包含字母、数字和下划线，并且必须以字母和下划线作为开头。

    <?php
	class foo {
	    public $bar = <<<EOT
	bar
	    EOT;
	}
	?>
#### Nowdoc 结构 ####
就像 heredoc 结构类似于双引号字符串，Nowdoc 结构是类似于单引号字符串的。Nowdoc 结构很象 heredoc 结构，但是 nowdoc 中不进行解析操作。这种结构很适合用于嵌入 PHP 代码或其它大段文本而无需对其中的特殊字符进行转义。与 SGML 的 <![CDATA[ ]]> 结构是用来声明大段的不用解析的文本类似，nowdoc 结构也有相同的特征。

一个 nowdoc 结构也用和 heredocs 结构一样的标记 <<<， 但是跟在后面的标识符要用单引号括起来，即 <<<'EOT'。Heredoc 结构的所有规则也同样适用于 nowdoc 结构，尤其是结束标识符的规则。

    <?php
	$str = <<<'EOD'
	Example of string
	spanning multiple lines
	using nowdoc syntax.
	EOD;
	
	/* 含有变量的更复杂的示例 */
	class foo
	{
	    public $foo;
	    public $bar;
	
	    function foo()
	    {
	        $this->foo = 'Foo';
	        $this->bar = array('Bar1', 'Bar2', 'Bar3');
	    }
	}
	
	$foo = new foo();
	$name = 'MyName';
	
	echo <<<'EOT'
	My name is "$name". I am printing some $foo->foo.
	Now, I am printing some {$foo->bar[1]}.
	This should not print a capital 'A': \x41
	EOT;
	?>
#### PHP中单引号和双引号的区别 ####
在PHP中，字符串的定义可以使用单引号，也可以使用双引号。但是必须使用同一种单或双引号来定义字符串，如：‘Hello"和“Hello'为非法的字符串定义。

定义字符串时，只有一种引号被视为定义符，即单引号或双引号。于是，如果一个字符串由双引号开始，那么只有双引号被分析器解析。这样，你就可以在双引号串中包含任何其他字符，甚至单引号。下面的引号串都是合法的： 

    $s = "I am a 'single quote string' inside a double quote string"; 

	$s = 'I am a "double quote string" inside a single quote string'; 
	
	$s = "I am a 'single quote string' inside a double quote string"; 
	
	$s = 'I am a "double quote string" inside a single quote string'; 

而串 "Why doesn't "this" work?" 则会被分为三段。如果在这个串中想要表示出双引号，则可以使用转义符"\"（反斜线），变成 "Why doesn't \"this\" work?" 即可。 

定义字符串变量时，PHP允许我们在双引号串中直接包含字串变量，我们可以发现下面的两个字串的处理结果是相同的。 

    $full_name = $first_name . ' ' . $last_name; 
	
	$full_name = "$first_name $last_name"; 

单引号串和双引号串在PHP中的处理是不相同的。**双引号串中的内容可以被解释而且替换，而单引号串中的内容总被认为是普通字符。**

一般情况下两者是通用的，但是” ” 双引号里面的字段会经过编译器解释，然后再当作HTML代码输出。‘ ‘ 单引号里面的不进行解释，直接输出。从字面意思上就可以看出，单引号比双引号要快了。所以如果内部只有纯字符串的时候,用单引号(速度快),内部有别的东西(如变量)的时候,用双号引更好点

**PHP引号使用原则：**

1. 字符串的值用引号
2. PHP中尽量用单引号,HTML代码全部用双引号
3. 在包含变量的时候,用双引号可以简化操作
4. 复杂的情况下用大括号包起来
5. PHP引号还有一个用处就是，有的时候需要用php生成文本文件，换行符\n需要用双引号才能好使，单引号则会直接把\n当成字符输出。

### Array 数组  ###
PHP 中的数组实际上是一个有序映射。映射是一种把 values 关联到 keys 的类型。此类型在很多方面做了优化，因此可以把它当成真正的数组，或列表（向量），散列表（是映射的一种实现），字典，集合，栈，队列以及更多可能性。由于数组元素的值也可以是另一个数组，树形结构和多维数组也是允许的。

    <?php
	$array = array(
	    "foo" => "bar",
	    "bar" => "foo",
	);
	
	// 自 PHP 5.4 起
	$array = [
	    "foo" => "bar",
	    "bar" => "foo",
	];
	?>
在PHP中, 数组是用一种HASH结构(**HashTable**)来实现的,(但是PHP手册上说“PHP中的数组实际上是一个**有序图**。图是一种把values映射到keys的类型。此类型在很多方面做了优化，因此可以把它当成真正的数组来使用，或列表（矢量），散列表（是图的一种实现），字典，集合，栈，队列以及更多可能性。因为可以用另一个PHP数组作为值，也可以很容易地模拟树。”) PHP使用了一些机制, 使得可以在O(1)的时间复杂度下实现数组的增删, 并同时支持线性遍历和随机访问. 

PHP中遍历数组的方式汇总：

1.foreach 

一个用来遍历数组中数据的最简单有效的方法。

    <?php
	$colors= array('red','blue','green','yellow');
	foreach ($colors as $color){
		echo "Do you like $color? <br />";
		}
	?>
2.while 

while() 通常和 list()，each()配合使用。

    <?php
	$colors= array('red','blue','green','yellow');
	while(list($key,$val)= each($colors)) {
		echo "Other list of $val.<br />";
		}
	?>
3.for()

    <?php
	$arr= array ("0"=> "zero","1"=> "one","2"=> "two");
	for ($i= 0;$i< count($arr); $i++){
		$str= $arr[$i];
		echo "the number is $str.<br />";
		}
	?>
4.key() 

key()函数返回input_array中位于当前指针位置的键元素。

    <?php
	$capitals= array("Ohio"=> "Columbus","Towa"=> "Des Moines","Arizona"=> "Phoenix");
	echo "<p>Can you name the capitals of these states?</p>";
	while($key= key($capitals)) {
		echo $key."<br />";
		next($capitals);
		//每个key()调用不会推进指针。为此要使用next()函数
		}
	?>
5.reset()

reset()函数用来将input_array的指针设置回数组的开始位置。如果需要在一个脚本中多次查看或处理同一个数组，就经常使用这个函数，另外这个函数还常用于排序结束时。

    <?php
	$colors= array('red','blue','green','yellow');
	foreach ($colorsas$color){
		echo "Do you like $color? <br />";
	}
	reset($colors);
	while(list($key,$val)= each($colors)) {
		echo "$key=> $val<br />";
	}
	?>
6.each()

each()函数返回输入数组当前键/值对，并将指针推进一个位置。返回的数组包含四个键，键0和key包含键名，而键1和value包含相应的数据。如果执行each()前指针位于数组末尾，则返回FALSE。

    <?php
	$capitals= array("Ohio"=> "Columbus","Towa"=> "Des Moines","Arizona"=> "Phoenix");
	$s1= each($capitals);
	print_r($s1);
	?>
7.current()，next()，prev()，end()

current()函数 返回位于targetArray数组当前指针位置的数组值。与next()、prev()、和end()函数不同，current()不移动指针。 

next()函数 返回紧接着放在当前数组指针的下一个位置的数组值。 

prev()函数 返回位于当前指针的前一个位置的数组值，如果指针本来就位于数组的第一个位置，则返回FALSE。 

end()函数 将指针移向targetArray的最后一个位置，并返回最后一个元素。

    <?php
	$fruits= array("apple","orange","banana");
	$fruit= current($fruits); //return "apple"
	echo $fruit."<br />";
	$fruit= next($fruits); //return "orange"
	echo $fruit."<br />";
	$fruit= prev($fruits); //return "apple"
	echo $fruit."<br />";
	$fruit= end($fruits); //return "banana"
	echo $fruit."<br />";
	?>

对数组遍历速度进行测试，测试的遍历方法为常用的三种for、while、foreach。经过反复多次测试，结果表明，对于遍历同样一个数组，foreach速度最快，最慢的则是while。从原理上来看，foreach是对数组副本进行操作（通过拷贝数组），而while则通过移动数组内部指标进行操作，一般逻辑下认为，while应该比foreach快（因为foreach在开始执行的时候首先把数组复制进去，而while直接移动内部指标。），但结果刚刚相反。原因应该是，foreach是PHP内部实现，而while是通用的循环结构。所以，在通常应用中foreach简单，而且效率高。在PHP5下，foreach还可以遍历类的属性。
## 对象 ##
自 PHP 5 起完全重写了对象模型以得到更佳性能和更多特性。PHP 5 中的新特性包括访问控制，抽象类和 final 类与方法，附加的魔术方法，接口，对象复制和类型约束。PHP 对待对象的方式与引用和句柄相同，即每个变量都持有对象的引用，而不是整个对象的拷贝。
### class ###
每个类的定义都以关键字 class 开头，后面跟着类名，后面跟着一对花括号，里面包含有类的属性与方法的定义。

类名可以是任何非 PHP 保留字的合法标签。一个合法类名以字母或下划线开头，后面跟着若干字母，数字或下划线。以正则表达式表示为：**[a-zA-Z_\x7f-\xff][a-zA-Z0-9_\x7f-\xff]\***。

一个类可以包含有属于自己的常量，变量（称为“属性”）以及函数（称为“方法”）。

    <?php
	class SimpleClass
	{
	    // property declaration
	    public $var = 'a default value';
	
	    // method declaration
	    public function displayVar() {
	        echo $this->var;
	    }
	}
	?>
当一个方法在类定义内部被调用时，有一个可用的伪变量 $this。$this 是一个到主叫对象的引用（通常是该方法所从属的对象，但如果是从第二个对象静态调用时也可能是另一个对象）。
### new  ###
要创建一个类的实例，必须使用 new 关键字。当创建新对象时该对象总是被赋值，除非该对象定义了构造函数并且在出错时抛出了一个异常。类应在被实例化之前定义（某些情况下则必须这样）。

如果在 new 之后跟着的是一个包含有类名的字符串，则该类的一个实例被创建。如果该类属于一个名字空间，则必须使用其完整名称。

    <?php
	$instance = new SimpleClass();
	
	// 也可以这样做：
	$className = 'Foo';
	$instance = new $className(); // Foo()
	?>
在类定义内部，可以用 new self 和 new parent 创建新对象。

当把一个对象已经创建的实例赋给一个新变量时，新变量会访问同一个实例，就和用该对象赋值一样。此行为和给函数传递入实例时一样。可以用克隆给一个已创建的对象建立一个新实例。
### extends  ###
一个类可以在声明中用 extends 关键字继承另一个类的方法和属性。PHP不支持多重继承，一个类只能继承一个基类。

被继承的方法和属性可以通过用同样的名字重新声明被覆盖。但是如果父类定义方法时使用了 final，则该方法不可被覆盖。可以通过 parent:: 来访问被覆盖的方法或属性。

当覆盖方法时，参数必须保持一致否则 PHP 将发出 E_STRICT 级别的错误信息。但构造函数例外，构造函数可在被覆盖时使用不同的参数。

    <?php
	class ExtendClass extends SimpleClass
	{
	    // Redefine the parent method
	    function displayVar()
	    {
	        echo "Extending class\n";
	        parent::displayVar();
	    }
	}
	
	$extended = new ExtendClass();
	$extended->displayVar();
	?>
### ::class ###
自 PHP 5.5 起，关键词 class 也可用于类名的解析。使用 ClassName::class 你可以获取一个字符串，包含了类 ClassName 的完全限定名称。这对使用了 命名空间 的类尤其有用。这个是作用域限定操作符，是用一个双冒号"::"表示，它用来置顶类中不同作用域的级别。左边是作用域,右边是访问作用域的成员。

    <?php
	namespace NS {
	    class ClassName {
	    }
	    
	    echo ClassName::class;
	}
	?>
以上例程会输出：

    NS\ClassName

在php中定义的作用域有self和parent两种（在php6中提供了static作用域）。

self:表示当前类的作用域，与this不同的是它不表示类的某个特定实例，在类之外的代码中不能使用self，而且它不能识别自己在继承中层次的位置。也就是说，**当在扩展类中使用self时，它调用的不是父类的方法，而是扩展类的重载的方法**。

parent：表示当前类父类的作用域，其余的跟self特性一样。

    <?php 
	class forasp {   
	    static $url="http://blog.csdn.net/abandonship";   
	    static $webname = "PHP学习之双冒号的用法";   
	    public function writeurl(){     
	        echo self::$url;//调用自己的内容   
	    }   
	    public function writewebname(){     
	        echo "测试子类调用父类内容";       } 
	}  
	class cn extends forasp {   
	    function father() {     
	        parent::wirtewebname();   
	    } 
	}  
	$a = new forasp();//实例化父类 
	$a->writeurl();//调用自身内容 
	$b = new cn(); 
	$b->writewebname();//调用父类内容 
	?>