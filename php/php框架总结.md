## tp 
### 如何理解TP中的单一入口文件？
ThinkPHP采用单一入口模式进行项目部署和访问，无论完成什么功能，一个项目都有一个统一（但不一定是唯一）的入口。应该说，所有项目都是从入口文件开始的，并且所有的项目的入口文件是类似的，入口文件中主要包括：
定义框架路径、项目路径和项目名称（可选）
定义调试模式和运行模式的相关常量（可选）
载入框架入口文件（必须）---动态加载框架实例

### ThinkPHP中的MVC分层是什么？
MVC 是一种将应用程序的逻辑层和表现层进行分离的方法。ThinkPHP 也是基于MVC设计模式的。MVC只是一个抽象的概念，并没有特别明确的规定，ThinkPHP中的MVC分层大致体现在：
模型（M）：模型的定义由Model类来完成。
控制器（C）：应用控制器（核心控制器App类）和Action控制器都承担了控制器的角色，Action控制器完成业务过程控制，而应用控制器负责调度控制。
视图（V）：由View类和模板文件组成，模板做到了100％分离，可以独立预览和制作。

### 全局函数
S
D
M
C
I


### ThinkPHP如何防止SQL注入？
如果不得已必须使用字符串查询条件，使用预处理机制；
使用字段类型检查、自动验证和自动完成机制等避免恶意数据的输入
开启数据字段类型验证，可以对数值数据类型做强制转换
sql操作能用Array操作就用Array

### 如何开启调试模式？调试模式有什么好处
任何错误信息和调试信息都会详细记录，便于调试。运行日志会更多，更细致
基于缓存的模版，表结构检测会失效

### ThinkPHP框架中D函数与M函数的区别是什么？
在实例化的过程中，经常使用D方法和M方法，这两个方法的区别在于M方法实例化模型无需用户为每个数据表定义模型类，如果D方法没有找到定义的模型类，则会自动调用M方法。
通俗一点说：
M实例化参数是数据库的表名。
D实例化的是你自己在Model文件夹下面建立的模型文件
### TP中系统变量有哪些？如何获取系统变量？
$this->方法名("变量名",["过滤方法"],["默认值"])

### TP中的URL模式有哪几种？默认是哪种？
ThinkPHP支持四种URL模式，可以通过设置URL_MODEL参数来定义，包括普通模式、PATHINFO、REWRITE和兼容模式。
默认模式为：PATHINFO模式，设置URL_MODEL 为1
普通模式：http://localhost/?m=home&c=user&a=login&var=value
PATHINFO模式：http://localhost/index.php/home/user/login?var=value 依然是有效的
REWRITE模式：http://localhost/home/user/login/var/value
入口文件同级新增
```
<IfModule mod_rewrite.c>
 RewriteEngine on
 RewriteCond %{REQUEST_FILENAME} !-d
 RewriteCond %{REQUEST_FILENAME} !-f
 RewriteRule ^(.*)$ index.php/$1 [QSA,PT,L]
</IfModule>
```


## ci 
### CodeIgniter最突出的特点是什么
重量极轻。它基于模型视图控制器（MVC）模式。
它具有全功能的数据库类和对多个平台的支持


## yii
###  Active Record
Active Record 提供了一个面向对象的接口， 用以访问和操作数据库中的数据。Active Record 类与数据库表关联， Active Record 实例对应于该表的一行， Active Record 实例的属性表示该行中特定列的值。 您可以访问 Active Record 属性并调用 Active Record 方法来访问和操作存储在数据库表中的数据， 而不用编写原始 SQL 语句。
更直观， 更不易出错

### 源码总结
https://segmentfault.com/a/1190000011823699
https://segmentfault.com/a/1190000010788354

## laveral

