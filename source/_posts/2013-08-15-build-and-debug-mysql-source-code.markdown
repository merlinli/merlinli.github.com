---
layout: post
title: "GDB 调试mysql cluster 代码"
date: 2013-08-15 14:40
comments: true
categories: mysql
---
	1.下载code
	2.编译，安装
    3.配置
    4.调试

##1.下载code

download link 
[http://dev.mysql.com/downloads/cluster/#downloads](http://dev.mysql.com/downloads/cluster/#downloads "mysql cluster soure code")
其实就是platform选择source code 就可以了。

这种方式只能下载最新的的code，如果你要下载特定的分支，要下载 Bazaar，这个是mysql的分支管理软件。
详细：  
[getting-started-with-bazaar-for-mysql.html](http://dev.mysql.com/tech-resources/articles/getting-started-with-bazaar-for-mysql.html "getting-started-with-bazaar-for-mysql")


##2.编译，安装
不同版本可能编译方式不同，所以最好找到编译的说明文件，代码下载好了在./BUILD目录下有个README 文件。  
你如果要调试的话，肯定是要用debug方式编译。  

对于 mysql-cluster-7.3.2版本   
编译

   >\#cd ./BUILD  
   >\#cmake -DWITH_DEBUG=1 -DCMAKE_INSTALL_PREFIX=/将来默认安装目录 /home/workspace/mysql-cluster-gpl-7.3.2/     
   **Note**：<br/>
             这个命令主要是生成makefile,如果缺少对应的工具自己安装，比如我新装的机器，缺少cmake，
             自己下个源码包编译，安装下就 可以，jdk的版本至少是要1.6.1。设定好$JAVA_HOME,并将之加入到$PATH.     
             WITH_DBUG=1                      表示是DEBUG编译  
			 CMAKE_INSTALL_PREFIX=  目录，     表示将来软件安装的默认目录
   
   >\#make  
   >......  
   >[100%] Built target my_safe_process   
   编译完之后最后一句  

安装
   >\#make install  
   如果你在cmake阶段没有指定CMAKE_INSTALL_PREFIX 的值，那它的默认安装目录就是/usr/local/mysql.
   这个目录会写死在各种脚本里，特别要注意。

##3.配置  
	# groupadd mysql
	# useradd -r -g mysql mysql
    cd 你的安装目录
    # chown -R mysql .
    # chgrp -R mysql .
    # scripts/mysql_install_db --user=mysql  //会在当前目录下生成默认配置文件my.cnf
	# chown -R root .
	# chown -R mysql data
	# chown mysql my.cnf  
事实上这些初始化步骤可以很清楚的在安装目录下INSTALL-BINARY里看到。
### 配置文件修改
	># vi my.cnf 
增加以下内容：  

 	basedir = /home/emenlii/debug-ndb
	datadir = /home/emenlii/debug-ndb/data
	socket = /home/emenlii/debug-ndb/run/mysql.sock
	log-error = /home/emenlii/debug-ndb/log/error.log
	log_slow_queries = /home/emenlii/debug-ndb/log/slow.log

 Note：  
	 如果你的安装目录跟我不一样，可以用以下的vim命令做个简单的替换。
	:%s/\/home\/mysql\/mysql/\/home\/emenlii\/debug-ndb/g
    这件话的意思是全文搜索 将字符串“/home/mysql/mysql” 替换为 “/home/emenlii/debug-ndb”
    因为我拷贝的是别人的配置文件，别人的安装目录是“/home/mysql/mysql”  
    “/home/emenlii/debug-ndb”是我的安装目录。   
    “\/” 第一个反斜杠是转义字符用。
    
  
	#mkdir log run tmp

##启动
#####1.直接启动  
	# ./bin/mysqld --basedir=/home/emenlii/debug-ndb/ --datadir=/home/emenlii/debug-ndb/data/ --user=mysql &  

或者
#####2.mysqld_safe
	#./bin/mysqld_safe&  
	ps -elf|grep mysql  
	4 S root      2383  4975  0  80   0 -  2927 wait   14:13 pts/0    00:00:00 /bin/sh ./bin/mysqld_safe  
	4 S mysql     2467  2383  0  80   0 - 122398 -     14:13 pts/0    00:00:00 /home/emenlii/debug-ndb/bin/mysqld --basedir=/home/emenlii/debug-ndb --datadir=/home/emenlii/debug-ndb/data --plugin-dir=/home/emenlii/debug-ndb/lib/plugin --user=mysql --log-error=/home/emenlii/debug-ndb/data/linux-50.err --pid-file=/home/emenlii/debug-ndb/data/linux-50.pid  

###打开mysql客户端
	#./bin/mysql
	mysql> show databases;	  
	+--------------------+   
	| Database           |  
	+--------------------+  
	| information_schema |  
	| mysql              |  
	| performance_schema |    
	| test               |  
	+--------------------+  
4 rows in set (0.00 sec)


##调试

	# ./bin/mysqladmin shutdown //先关闭进程
	# gdb -args /home/emenlii/debug-ndb/bin/mysqld --basedir=/home/emenlii/debug-ndb/ --datadir=/home/emenlii/debug-ndb/data/ --user=mysql  
    然后加断点，然后就可以在GDB里 run, 另开一个窗口操作mysql客户端



    
