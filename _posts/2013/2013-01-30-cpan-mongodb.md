---
layout: post
category : tech
title:  "perl MongoDB 连接数据库"
tagline: "nosql"
tags : ["perl", "cpan", "mongodb", "nosql" ] 
---
{% include JB/setup %}

## 查询 MongoDB 

{% highlight perl %}
#!/usr/bin/perl
use strict;
use warnings;

use MongoDB;

my $db_name    = "test_db";
my $table_name = "test_table";
my $connection = MongoDB::Connection->new(
host          => 'xxx.xxx.xxx.xxx',
port          => 27017,
username      => "some_user",
password      => "some_passwd",
query_timeout => 300000,
slaveok       => 1,
db_name       => $db_name,
);

my $database   = $connection->get_database($db_name);
my $collection = $database->get_collection($table_name);
my $cursor       = $collection->find( { somecol => $someval } );

while(my $o = $cursor->next){
print Dumper($o);
}
{% endhighlight %}

## 向 MongoDB 插入数据

- Perl : 5.16
- MongoDB : 2.2
- Perl CPAN Module：MongoDB-0.503.2

用perl脚本往MongoDB里插数据，如果数据是多层嵌套hash，基本上直接崩溃。

插入程序参考：https://metacpan.org/module/MongoDB

解决方案：在perl程序中将数据转换为json字符串，写成文本文件，再调用mongoimport指令导入数据库。

mongoimport用法：http://docs.mongodb.org/manual/reference/mongoimport/

## 连接失败，显示 can't get db response

- perl：5.16
- mongodb server : 2.2
- MongoDB cpan module: 0.503.3

出错信息： can't get db response, not connected

解决方案：https://groups.google.com/forum/?fromgroups=#!topic/mongodb-user/BxHM3cM4q5Q

下载MongoDB的cpan模块源代码，修改其中mongo_link.c的一行内容：

tv.tv_sec = 1;
改成
tv.tv_sec = timeout;

重新编译安装即可。 