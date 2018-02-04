# 介绍

## 简介

一套完整的基于Docker的PHP开发环境。

包含了预先打包的Docker镜像，所有预先配置都是为了提供一个完美的PHP开发环境。

`Laradock`是在`laravel`社区众所周知的，因为这个项目最开始只关注在`Docker`上运行的`laravel`项目。后来，由于PHP社区的大量使用，它开始支持比如`Symfony`、`CodeIgniter`、`WordPress`、`Drupal`等其他的PHP项目。

![](https://s19.postimg.org/jblfytw9f/laradock-logo.jpg)

## 快速概览

让我们来看一下用它安装`NGINX`, `PHP`, `Composer`, `MySQL`, `Redis` and `Beanstalkd`是多么容易：

- 克隆`Laradock`到你的PHP项目中:

```shell
git clone https://github.com/Laradock/laradock.git
```

- 进入laradock文件夹并重命名`env-example`为`.env`:

```shell
cp env-example .env
```

- 运行你的容器:

```shell
docker-compose up -d nginx mysql redis beanstalkd
```

 - 打开你项目里的`.env`文件并设置以下内容:

```shell
DB_HOST=mysql
REDIS_HOST=redis
QUEUE_HOST=beanstalkd
```

- 打开浏览器并访问本地主机: `http://localhost`.

```shell
That's it! enjoy :)
```


## 特点

- 轻松切换PHP版本: 7.1, 7.0, 5.6...
- 选择你最喜欢的数据库引擎: MySQL, Postgres, MariaDB...
- 运行你自己的软件组合: Memcached, HHVM, Beanstalkd...
- 每个软件都运行在一个单独的容器中: PHP-FPM, NGINX, PHP-CLI...
- 通过简单的编辑 `Dockerfile`很容易定制任何容器
- 所有镜像都继承自官方的基础镜像 (可信任的基础镜像)。
- 预先配置在您的根目录下的NGINX可以托管任何代码。
- 每个项目都可以使用laradock，或者所有项目也可以共用一个laradock。
- 可以很容易的使用环境变量安装/删除容器中的软件。
- 干净、结构很好的Dockerfiles (`Dockerfile`)。
- 最新版的Docker Compose文件 (`docker-compose`)。
- 一切都是可见、可编辑的。
- 快速的镜像构建。
- 每周会有更多更新。


## 支持的软件

在秉承Docker推动的关注分离原则的同时，Laradock在各自的容器中(Container)独立运行各自的软件，你可以像任何容器一样打开/关闭多个实例，而不用担心配置，一切都很有魅力。


- **数据库引擎:**
MySQL - MariaDB - Percona - MongoDB - Neo4j - RethinkDB - MSSQL - PostgreSQL - Postgres-PostGIS.
- **数据库管理:**
PhpMyAdmin - Adminer - PgAdmin
- **缓存引擎:**
Redis - Memcached - Aerospike
- **PHP 服务器:**
NGINX - Apache2 - Caddy
- **PHP 编译器:**
PHP FPM - HHVM
- **消息队列:**
Beanstalkd - RabbitMQ - PHP Worker
- **队列管理:**
Beanstalkd Console - RabbitMQ Console
- **随机 工具:**
HAProxy - Certbot - Blackfire - Selenium - Jenkins - ElasticSearch - Kibana - Grafana - Mailhog - MailDev - Minio - Varnish - Swoole - Laravel Echo...

作为一个开发环境，Laradock引入了**Workspace**镜像，它包含一套丰富的有用的工具，所有工具都用上预先配置好的，几乎可以与所有你选择的容器或者工具组合起来一起工作。

**Workspace 镜像工具**

PHP CLI - Composer - Git - Linuxbrew - Node - V8JS - Gulp - SQLite - xDebug - Envoy - Deployer - Vim - Yarn - SOAP - Drush...

你可以从`.env`文件中选择要安装的在工作区的容器和其它容器中的工具。


> 如果你修改`docker-compose.yml`, `.env` 或者任何 `dockerfile` 文件, 你必须重新构建你的容器, 在运行实例中查看这些修改的效果。


如果您在列表中找不到您的软件，请自行构建并提交。贡献欢迎:)



## 赞助商

通过成为赞助商来支持这个项目。

您的logo将会显示在[github 仓库](https://github.com/laradock/laradock/) 的索引页和 [文档](http://laradock.io/) 主页， 并链接到您的网站. [[成为赞助商](https://opencollective.com/laradock#sponsor)]

<a href="https://opencollective.com/laradock/sponsor/0/website" target="_blank"><img src="https://opencollective.com/laradock/sponsor/0/avatar.svg"></a>
<a href="https://opencollective.com/laradock/sponsor/1/website" target="_blank"><img src="https://opencollective.com/laradock/sponsor/1/avatar.svg"></a>
<a href="https://opencollective.com/laradock/sponsor/2/website" target="_blank"><img src="https://opencollective.com/laradock/sponsor/2/avatar.svg"></a>
<a href="https://opencollective.com/laradock/sponsor/3/website" target="_blank"><img src="https://opencollective.com/laradock/sponsor/3/avatar.svg"></a>
<a href="https://opencollective.com/laradock/sponsor/4/website" target="_blank"><img src="https://opencollective.com/laradock/sponsor/4/avatar.svg"></a>
<a href="https://opencollective.com/laradock/sponsor/5/website" target="_blank"><img src="https://opencollective.com/laradock/sponsor/5/avatar.svg"></a>
<a href="https://opencollective.com/laradock/sponsor/6/website" target="_blank"><img src="https://opencollective.com/laradock/sponsor/6/avatar.svg"></a>
<a href="https://opencollective.com/laradock/sponsor/7/website" target="_blank"><img src="https://opencollective.com/laradock/sponsor/7/avatar.svg"></a>
<a href="https://opencollective.com/laradock/sponsor/8/website" target="_blank"><img src="https://opencollective.com/laradock/sponsor/8/avatar.svg"></a>
<a href="https://opencollective.com/laradock/sponsor/9/website" target="_blank"><img src="https://opencollective.com/laradock/sponsor/9/avatar.svg"></a>


## 什么是Docker

[Docker](https://www.docker.com) 是一个开发、运输、运行应用程序的开放平台。

Docker可以使你将应用程序和你的基础架构(设备)分离，因此你可以快速的交付你软件。

使用Docker，你可以管理应用程序一样管理你的基础架构，通过使用Docker的methodologies来快速的传输、测试和部署代码，可以显著的缩短编写代码和生成环境中运行的代码之间的延迟。



## 为什么是Docker而不是Vagrant

[Vagrant](https://www.vagrantup.com) 可以在进分钟内创建虚拟机， 而 Docker 是在几秒中内创建虚拟容器。

Docker提供**轻量级**的虚拟容器，它们共享相同的内核，并允许安全的执行独立的进程，而不像Vagrant一样提供完整的虚拟机。

除了速度之外，Docker还提供了很多Vagrant无法实现的功能。

最重要的是，Docker可以运行在开发和生产（任何地方都可以做到相同的环境），而Vagrant仅仅只是为了开发而设计（因此您必须每次在生产环境中重新配置您的服务器）



## 演示视频

没有什么比演示视频更好啦：

- Laradock v5.* (should be next!)
- Laradock [v4.*](https://www.youtube.com/watch?v=TQii1jDa96Y)
- Laradock [v2.*](https://www.youtube.com/watch?v=-DamFMczwDA)
- Laradock [v0.3](https://www.youtube.com/watch?v=jGkyO6Is_aI)
- Laradock [v0.1](https://www.youtube.com/watch?v=3YQsHe6oF80)


## 和我们交流

欢迎您加入我们的Gitter聊天室。

[![Gitter](https://badges.gitter.im/Laradock/laradock.svg)](https://gitter.im/Laradock/laradock?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge)



## 捐赠

> 通过 [捐助](http://laradock.io/contributing) 来帮助保持项目的发展。

> 提前致谢。

通过[Paypal](https://www.paypal.me/mzalt)直接捐赠

[![Donate](https://img.shields.io/badge/Donate-PayPal-green.svg)](https://www.paypal.me/mzalt) 

或者成为[公开集体](https://opencollective.com/laradock#backer)的支持者。

<a href="https://opencollective.com/laradock#backers" target="_blank"><img src="https://opencollective.com/laradock/backers.svg?width=890"></a>

或者通过 [Beerpay](https://beerpay.io/laradock/laradock) 表示支持

[![Beerpay](https://beerpay.io/laradock/laradock/badge.svg?style=flat)](https://beerpay.io/laradock/laradock)
