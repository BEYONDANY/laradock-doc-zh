# 入门

## 要求

- [Git](https://git-scm.com/downloads)
- [Docker](https://www.docker.com/products/docker/) `>= 1.12`



## 安装

选择最适合你的需求进行设置

- [单个项目的设置](#单个项目的设置)
  - [已经有一个PHP项目](#已经有一个php项目)
  - [还没有一个PHP项目](#还没有一个php项目)
- [多个项目的设置](#多个项目的设置)


### 单个项目的设置
> (如果你想未灭个项目单独配置Docker环境，按照下面这些步骤)


#### 已经有一个PHP项目

1 - 将laradock克隆到你项目的根目录:

```bash
git submodule add https://github.com/Laradock/laradock.git
```

注意: 如果你还没有为你的项目使用Git，你可以使用 `git clone`代替 `git submodule `。

*记录你的laradock的变化同时保持更新laradock到你的项目 [查看这些文档](/documentation/#keep-track-of-your-laradock-changes)*


你的文件夹结构应该如下所示:

```
+ project-a
  + laradock-a
+ project-b
  + laradock-b
```

*(如果你想为每个项目运行单独的laradock，将laradock文件夹重命名为每个项目中的唯一名称很重要).*

> **现在跳转到[用法](#用法)部分.**

### 还没有一个PHP项目

- 将repository克隆到你机器的任意位置:

```bash
git clone https://github.com/laradock/laradock.git
```

你的文件夹结构应该如下所示

```
+ laradock
+ project-z
```

- 编辑你的web服务器站点配置

我们需要去执行[用法](#用法)部分的第一步来实现这一点

```
cp env-example .env
```

在顶部, 修改 `APPLICATION`变量为你的项目路径.

```
APPLICATION=../project-z/
```

确保使用你的项目文件夹名称将`project-z`进行替换

> **现在跳转到[用法](#用法)部分**


### 多个项目的设置

> (如果你想为你的所有项目使用单个的Docker环境，请遵循以下步骤)

- 在你的机器上的任意位置克隆repository(类似于[上面的还没有一个PHP项目](#还没有一个PHP项目)):

```bash
git clone https://github.com/laradock/laradock.git
```

你的文件夹结构应该如下所示:

```
+ laradock
+ project-1
+ project-2
```

- 进入`nginx/sites`为访问的不同的域名创建配置文件去指向不同的项目目录

默认情况下laradock包含`app.conf.example`, `laravel.conf.example` 和 `symfony.conf.example` 作为工作样本

- 修改默认名称为`*.conf`:

你可以根据需要重命名配置文件、项目文件夹和域名，只要确保`root`在配置文件中指向正确的文件夹名称即可。

4 - 在**hosts**文件夹中增加域名

```
127.0.0.1  project-1.test
127.0.0.1  project-2.test
...

```
如果你使用的是Chrome 63或者更高版本进行开发的,请勿使用`.dev`的域名 [为什么?](https://laravel-news.com/chrome-63-now-forces-dev-domains-https). 使用 `.localhost`, `.invalid`, `.test`,  `.example`取代它。

> **现在跳转到[用法](#用法)部分**



## 用法

**开始阅读之前:**

如果你使用的是 **Docker Toolbox** (VM), 请执行以下操作中的任意一个:

- 升级Mac/Windows[Native](https://www.docker.com/products/docker)(推荐). 检查[升级 Laradock](/documentation/#upgrading-laradock)
- 使用 Laradock v3.\*. Visit the [Laradock-ToolBox](https://github.com/laradock/laradock/tree/Laradock-ToolBox) branch. *(outdated)*


我们推荐使用比1.13更新的Docker版本


>**警告:** 如果你使用旧版的laradock，强烈推荐你所使用的容器，你需要[看怎样重新构建容器](#Build-Re-build-Containers)以尽量避免错误


- 进入laradock文件夹并复制`env-example` to `.env`

```shell
cp env-example .env
```

你可以编辑`.env`文件去选择你想在环境中安装的软件，你始终可以参考`docker-compose.yml`文件去看如何使用这些变量。


根据主机的操作系统，您可能需要更改给定的值 `COMPOSE_FILE`的值， 当你在MacOS上运行laradock时要使用的文件分隔符是`:`， 当你在Windows环境上运行laradock时，必须使用 `;`来作为多个文件的分隔符。

- 运行`docker-compose`来构建环境

在这个例子中我们将看到如何运行Nginx(web服务器)和MySQL(数据库引擎)来托管一个PHP脚本

```bash
docker-compose up -d nginx mysql
```

**注意**: 大多数情况下 `workspace` 和 `php-fpm` 会自动运行, 所以不需要再`up`命令中指定它们， 如果你找不到他们，那么你需要这样指定它们: `docker-compose up -d nginx php-fpm mysql workspace`。


你可以从[这个列表](http://laradock.io/introduction/#supported-software-images)中选择自己的容器组合

*(请注意，有时我们忘记更新文档，请检查`docker-compose.yml`文件以查看所有可用容器的更新列表)*


- 进入Workspace容器, 执行比如(Artisan, Composer, PHPUnit, Gulp, ...)等命令

```bash
docker-compose exec workspace bash
```

*另外, 针对 Windows PowerShell 用户: 执行以下命令以进入任何正在运行的容器:*

```bash
docker exec -it {workspace-container-id} bash
```

**注意:** 你可以添加`--user=laradock`来创建主机的用户. 比如: 

```shell
docker-compose exec --user=laradock workspace bash
```

*您可以从`.env`文件中更改PUID (用户标识) 和 PGID (组标识) 变量*


- 更新您的项目配置以使用数据库主机

打开你PHP项目的`.env`文件或者你正在读取的配置文件, 并将数据库配置`DB_HOST`设置为`mysql`:

```env
DB_HOST=mysql
```

*如果你想将laravel安装为PHP项目, 请参阅 [如和在Docker容器中安装laravel](#Install-Laravel).*


- 打开浏览器并访问您的本地主机地址`http://localhost/`， 如果你按照多个项目设置，你可以访问 `http://project-1.test/` 和 `http://project-2.test/`。
