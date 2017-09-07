### tencent server

``123.206.196.239
    Ubuntu 

###  压缩命令
 
	[root@linux ~]# tar -cvf /tmp/etc.tar /etc <==仅打包，不压缩！
	[root@linux ~]# tar -zcvf /tmp/etc.tar.gz /etc <==打包后，以 gzip 压缩
	[root@linux ~]# tar -jcvf /tmp/etc.tar.bz2 /etc <==打包后，以 bzip2 压缩
	
	将 /tmp/etc.tar.gz 文件解压缩在 /usr/local/src 底下
	[root@linux ~]# cd /usr/local/src
	[root@linux src]# tar -zxvf /tmp/etc.tar.gz

### 文件
	列出文件 Linux是ls，Win是dir
	复制 cp
	移动 mv
	删除 rm

### 进程 ###
	top 命令查看资源占用
	ps -A 查看后台进程
	nohup **** & 后台挂起