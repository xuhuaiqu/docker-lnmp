
DLNMP
=============

> 通过docker compose 根据官方镜像构建的环境 本项目适用于mac apple芯片版本
> PHP7.0.33 + Mysql最新版本 + Nginx最新版本 + Redis最新版本 + elasticsearch7.13.1



目录结构介绍
---------

	├── db
	│   ├── elasticsearch           elasticsearch数据库
	│   │   ├── conf          		elasticsearch配置文件
	│   │   ├── data				elasticsearch数据库表数据存放的位置
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
	│   │       └── php7.0.33.ini	php的配置文件
	│   └── web
	│       └── nginx
	└── tool
	    ├── app 					相当于/usr/work/tool/app 存放代码目录
	    ├── data
	    ├── log						日志目录
	    └── webserver 				nginx所在目录
	        ├── conf					nginx配置文件.conf
	        └── logs					nginx日志

安装docker和docker compose
-------------------------

Mac下直接到官网下载Docker.dmg 自带了docker-compose命令 

>注意docker低版本需要当前根目录到Preference->File Sharing 否则会出现Mounts denied权限不足



### docker compose 安装部署环境

下载当前库文件

1.进入上面下载完成后的文件夹，打开 `docker-compose.yml`

1.1更改mysql的密码：
```
- MYSQL_ROOT_PASSWORD=yourpasswd
```

1.2更改redis的密码：

```
打开文件：`./db/redis/etc/redis-password`,更改里面的redis密码即可。
```


2.构建：

启动docker

```
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



### 导入测试数据


1.测试数据:

1.1安装mysql数据库的测试数据 推荐使用mysqlworkbench 如果您使用的是命令行,安装mysql数据库的测试数据 将您的sql文件放在./db/mysql/example_db目录下


并在根目录(docker-compose.yml文件所在目录)下执行，进入mysql的容器

```
docker-compose exec mysql bash
```

执行`mysql -uroot -p` 进入mysql

```
use db;
source /var/example_db/mysql.sql #替换成你自己的sql文件名称
exit
```

`exit`，退出容器,回到宿主主机


### 常见问题
1.[Docker-compose up failing because "port is already allocated"](https://github.com/docker/compose/issues/4950)



### 参考资料
[yii2 fecshop docker](https://github.com/fecshop/yii2_fecshop_docker)

[Docker — 从入门到实践](https://yeasy.gitbooks.io/docker_practice)

[Docker LNMP环境搭建](https://www.awaimai.com/2120.html)
