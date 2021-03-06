# mysql

## 优化
> https://www.cnblogs.com/yuyangphpweibo/p/9044374.html

## php 之pdo
```
<?php
$dbh = new PDO('mysql:host=localhost;dbname=test', $user, $pass, array(
    PDO::ATTR_PERSISTENT => true
));
?>
```

## pdo的生命周期
很多 web 应用程序通过使用到数据库服务的持久连接获得好处。持久连接在脚本结束后不会被关闭，且被缓存，当另一个使用相同凭证的脚本连接请求时被重用。持久连接缓存可以避免每次脚本需要与数据库回话时建立一个新连接的开销，从而让 web 应用程序更快

1. 官方所说的脚本结束，在fpm模式下就是指一次客户端请求的结束。另一个使用相同凭证的脚本也就可以对应成另一个使用相同数据库连接凭证的客户端请求。首先我们要知道，这两次客户端的请求是根据fpm-workers的空闲情况，被分配给某个worker去执行的，所以两次请求被分配到同一个worker的可能性很低。接着，我们阐明下面的情景。
2. 开启持久连接之后，数据库连接是被缓存于fpm进程之中的。如果某个fpm-worker进程中已经缓存了持久连接，此时可能出现如下两种情况：
当脚本中再次执行带 ATTR_PERSISTENT 参数的pdo连接时，会复用之前的连接，而不会产生新的连接。
当脚本中再次执行不带 ATTR_PERSISTENT 参数的pdo连接时，还会再次产生一个新的数据库连接。

## sharding
https://www.cnblogs.com/f-ck-need-u/p/9388407.html

分库--》垂直拆分--》水平拆分--》分区-》扩容

