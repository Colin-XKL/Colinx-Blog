---
title: docker compose的错误使用姿势
date: 2022-05-11
description: 这篇文章记录几个docker compose使用过程中几个难以察觉的错误使用姿势。1. 变量值使用`@`符号开头 2. 使用6000或6666端口
categories:
- 技术
- 指南
tags:
- YAML
- Linux
- Docker
- docker-compose
---



<!-- # docker compose 的错误使用姿势 -->

这篇文章记录几个docker compose使用过程中几个难以察觉的错误使用姿势

## 1. 变量值使用`@`符号开头

我在RSS MAN X的docker-compose配置文件中，huignn的环境变量出来时是这样写的

```yaml
environment:
	- DATABASE_ADAPTER=postgresql
	- DATABASE_HOST=database.huginn
	- DATABASE_USERNAME=postgres
	- DATABASE_PASSWORD=@pass_for_DB
	- DATABASE_PORT=5432
```



自己使用的时候没发现什么问题，别人从零创建容器的时候报错了。刚开始排查了很久，后面发现我默认密码这样写其实是有问题的。

上面的使用列表来描述环境变量在docker-compose解析的时候没有任何报错信息。但是使用下面这种key-value的map的形式时，`docker-compose up`命令报错了，说YAML解析时出现问题

```yaml
environment:
	DATABASE_ADAPTER: postgresql
	DATABASE_HOST: database.huginn
	DATABASE_USERNAME: postgres
	DATABASE_PASSWORD: @pass_for_DB
	DATABASE_PORT: 5432
```



```
yaml: line 17: found character that cannot start any token
```

删除前导`@`符号则一切正常了。

```yaml
DATABASE_PASSWORD: pass_for_DB
```

后续测试了几次到底是什么原因导致的

```yaml
# error
DATABASE_PASSWORD: @pass_for_DB	

# correct
DATABASE_PASSWORD: pass_for_DB

# correct
DATABASE_PASSWORD: ${DB_Password:-'pass_for_DB'}
## in .env
DB_Password=www_123

# correct
DATABASE_PASSWORD: ${DB_Password:-'@pass_for_DB'}
## in .env
DB_Password=www_123

# error
DATABASE_PASSWORD: ${DB_Password:-'@pass_for_DB'}
## no .env

# error
DATABASE_PASSWORD: ${DB_Password:-'@pass_for_DB'}
## in .env
DB_Password=@www_123

# correct
DATABASE_PASSWORD: ${DB_Password:-'@pass_for_DB'}
## in .env
DB_Password=A@www_123
```

最终发现无论是否使用docker-compose变量替换，只要最终设定值的时候，包含前导@符号，最终一定会失败。Huginn的这边的报错为解析容器内的一个YAML文件时失败

```
huginn1     | foreman stderr | 
huginn1     | Caused by:
huginn1     | foreman stderr | Psych::SyntaxError: (<unknown>): found character that cannot start any token while scanning for the next token at line 8 column 13
huginn1     | foreman stderr | /app/vendor/bundle/ruby/2.6.0/gems/railties-6.0.4.4/lib/rails/application/configuration.rb:232:in `database_configuration'
```

后面我又进入容器查看了下容器内部读取到的环境变量到底是不是预期的值，发现了问题所在

能正常使用的情况

```shell
~ » sudo docker exec -it huginn1 /bin/bash
default@3b652a8413fb:~$ echo $DATABASE_HOST
database.huginn
default@3b652a8413fb:~$ echo $DATABASE_PASSWORD
A@www_123
```

出错的情况

```shell
~ » sudo docker exec -it huginn1 /bin/bash
default@42e19e692b24:~$ echo $DATABASE_HOST
database.huginn
default@42e19e692b24:~$ echo $DATABASE_PASSWORD
'@pass_for_DB'
```

可以看到，如果以`@`符号开头，最终的环境变量会加上引号，这对于某些应用来说会产生预期之外的结果，如此处测试的Huginn。





## 2. 使用6000或6666端口

这也是一定挺特殊的场景，虽然与docker compose关系不大，但是难免会遇到。在映射端口的时候有一个很坑的地方，如果你要部署一个web服务，然后映射的时候填写的是`6000`端口或者是`6666`端口，你会发现无论怎么重启容器、更改容器设置、更换Edge/Chrome，浏览器就是访问不了页面。

实在折腾地快疯了，点开Chrome的错误详情，才发现这么一句

 ***ERR_UNSAFE_PORT***

一查才发现，Chrome等浏览器会限制部分特殊端口的网页访问，这些端口一般都已经有特殊的作用。

https://www.jianshu.com/p/40f79f584eae

这里列出1000以上的部分会有相关问题的端口，别的不说，感觉`6000`和`6666`还是很容易一不小心就会设置的端口。。。。

```c
  2049, // nfs
  3659, // apple-sasl / PasswordServer
  4045, // lockd
  6000, // X11
  6665, // Alternate IRC
  6666, // Alternate IRC
  6667, // Standard IRC
  6668, // Alternate IRC
  6669, // Alternate IRC
```

Chrome、FireFox等都会有类似的行为，换多少浏览器、换多少版本可能也都不行。。。

根据网友描述，部分特殊情况下，其他可能会被拦截的端口也包括`8080`，`8443`，`10080`。

总之，日常映射端口还是选取`10000`以上的高位端口吧，避免不必要的麻烦：）