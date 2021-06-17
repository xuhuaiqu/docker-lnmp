
LNMP Docker Compose
=============


> 通过docker compose 根据官方镜像构建的环境 


目录结构介绍
---------

	├── db
	│   ├── elasticsearch           elasticsearch数据库
	│   │   ├── conf          		elasticsearch配置文件
	│   │   ├── data				elasticsearch数据库表数据存放的位置
	│   │   └── log
	│   ├── mysql             		mysql数据库
	│   │   ├── conf.d          	mysql 配置文件
	│   │   ├── data				mysql数据库表数据存放的位置
	│   │   ├── example_db			php的mysql示例数据
	│   │   └── log
	│   └── redis				
	│       ├── data
	│       └── etc
	│           ├── redis-password 	redis密码配置
	│           └── redis.conf 		redis配置文件
	├── docker-compose.yml			docker compose配置文件
	├── services
	│   ├── php
	│   │   ├── docker
	│   │   │   ├── Dockerfile		php镜像构建的dockerfile文件
	│   │   │   └── extension		PHP拓展目录在此添加你自己的拓展
	│   │   └── etc
	│   │       └── php7.0.26.ini	php的配置文件
	│   └── web
	│       └── nginx
	└── tool
	    ├── app 					相当于/usr/work/tool/app 存放代码目录
	    ├── conf
	    │   └── os_order 			相当于/usr/work/tool/conf 存放配置文件
	    ├── data
	    ├── framework 				框架目录
	    │   └── yii-1.1.13.e9e4a0
	    ├── log						日志目录 Yii_Log记录
	    │   └── log_data
	    ├── phplib					PHP Zrb常用库
	    ├── webroot					配置index.php的地方
	    │   └── os_order
	    └── webserver 				nginx所在目录
	        ├── conf					nginx配置文件.conf
	        ├── html
	        └── logs					nginx日志

安装docker和docker compose
-------------------------

Mac下直接到官网下载Docker.dmg 自带了docker-compose命令 

>注意docker需要当前根目录到Preference->File Sharing 否则会出现Mounts denied权限不足



### docker compose 安装部署环境

下载当前库文件

1.进入上面下载完成后的文件夹 zrb\_php\_docker，打开 `docker-compose.yml`

1.1更改mysql的密码：
```
- MYSQL_ROOT_PASSWORD=yourpasswd
```

1.2更改redis的密码：

```
打开文件：`./db/redis/etc/redis-password`,更改里面的redis密码即可。
```

mysql和redis的密码要记住，后面配置要用到。


2.构建：

启动docker

```
service docker start
```

> 第一次构建需要下载环境，时间会比较长，除了下载docker中心的镜像，还要构建镜像
> 看网速，如果用阿里云，15分钟差不多完成，使用下面的命令构建环境

```
chmod 755 /usr/local/bin/docker-compose
docker-compose build
```


完成后，运行：

```
docker-compose up  // 按下ctrl+c退出停止。
```

后台运行：（守护进程的方式）

```
docker-compose up -d
```

查看compose启动的各个容器的状态：

```
docker-compose ps
```

进入某个容器,譬如php：

```
docker-compose exec php bash
```

退出某个容器

```
exit
```


停止 docker compose启动的容器：

```
docker-compose stop
```

下面是一些简单说明


> 对于docker ，一定要切记，docker不是虚拟机！
> 每一个服务，对应一个docker 容器，譬如mysql
> 一个容器，php一个容器，redis一个容器
> 每一个容器的数据和配置文件都是在宿主主机上面，通过`volumes`
> 挂载到容器的相应文件夹中，（我们在`./docker-compose.yml`
> 配置文件中的`volumes`做了映射）
> 
> 因此，对于docker 容器，里面涉及到存储的部分，都应该通过
> 挂载的方式映射到宿主机上面，而不是在容器里面。

`宿主机`: 就是您的主机

`容器主机`：就是docker容器虚拟的主机。



### 配置php

1.添加域名hosts映射（也就是用ip映射的方式弄假域名），本机（浏览器所在的电脑，也就是您的window本机），添加host(打开etc\hosts，添加如下代码,如果是其他IP，将 127.0.0.1 替换成其他IP即可。)

```
127.0.0.1 os-vdong.xu.local

```
1.1在./tool/webserver/conf/vhost/server.conf新增您自己的域名 如：

	server {
	    listen 80;
	    server_name os-pinjamyuk.xu.local os-vdong.xu.local os-cashcash.xu.local os-pesoloan.xu.local;
	    # return 404;
	    #set_real_ip_from 127.0.0.1/32;
	    real_ip_header X-Forwarded-For;
	    charset UTF-8;
	
	    include ./vhost/modules/os_order;
	}
在./tool/webserver/conf/vhost/modules下面新增os_order文件内容为

	location /os_order/ {
	    root           /usr/work/tool/webroot/;
	    fastcgi_pass   php:9000;	#注意这里和线上不一样
	    fastcgi_index  index.html;
	    rewrite  ^/os_order/([^/]*)/?(/[^\?]*)?((\?.*)?)$  /os_order/index.php$1$2$3  break;
	    fastcgi_param  SCRIPT_FILENAME    $document_root$fastcgi_script_name;
	    fastcgi_split_path_info ^(.+\.php)(.*)$;
	    fastcgi_param PATH_INFO $fastcgi_path_info;
	    include        fastcgi_params;
	}


2.配置代码的配置文件

2.1数据库配置：
在./tool/conf下新建你自己的代码配置文件夹 如os_order配置文件ConfigBase.php示例

	'os_order' => array(
	            'connectionString' => 'mysql:host=mysql;port=3306;dbname=os_order',
	            'username' => 'root',
	            'password' => '123456', #你自己的数据库密码
	            'class' => 'CDbConnection',
	        ),



> 注意：修改数据库配置文件的时候，不要将各个数据库的host修改成`127.0.0.1`，使用默认配置的host即可（docker会通过host映射到相应的容器内网ip，各个容器是隔离的，各个容器的内网ip不是127.0.0.1）,否则，将会出现无法连接的问题



3.测试数据:

3.1安装mysql数据库的测试数据 推荐使用mysqlworkbench 如果您使用的是命令行,安装mysql数据库的测试数据 将您的sql文件放在./db/mysql/example_db目录下


并在根目录(docker-compose.yml文件所在目录)下执行，进入mysql的容器

```
docker-compose exec mysql bash
```

执行`mysql -uroot -p` 进入mysql

```
use php;
source /var/example_db/mysql.sql #替换成你自己的sql文件名称
exit
```

`exit`，退出容器,回到宿主主机


```
docker-compose stop
docker-compose up -d
```

然后就可以访问了
### 常见问题
1.[Docker-compose up failing because "port is already allocated"](https://github.com/docker/compose/issues/4950)



### 参考资料
[yii2 fecshop docker](https://github.com/fecshop/yii2_fecshop_docker)

[Docker — 从入门到实践](https://yeasy.gitbooks.io/docker_practice)

[Docker LNMP环境搭建](https://www.awaimai.com/2120.html)# dlnmp
