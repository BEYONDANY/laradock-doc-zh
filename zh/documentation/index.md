# 文档


## 列出正在运行的容器

```bash
docker ps
```
如果你只想看当前这个项目的容器，你也可以执行下面这个命令：

```bash
docker-compose ps
```


## 关闭所有正在运行的容器
```bash
docker-compose stop
```

停止单个容器:

```bash
docker-compose stop {container-name}
```


## 删除所有现有容器
```bash
docker-compose down
```


## 通过命令进入容器

### 1.首先使用`docker ps`列出当前正在运行的容器：

### 2.使用下面的命令进入任意容器：

```bash
docker-compose exec {container-name} bash
```

*例如: 进入MySQL容器*

```bash
docker-compose exec mysql bash
```

*例如: 进入MySQL并在MySQL容器中使用命令提示符*

```bash
docker-compose exec mysql mysql -u homestead -psecret
```

### 3.输入`exit`退出容器



## 编辑容器的默认配置

打开`docker-compose.yml`修改任何你想修改的配置。

栗子:

修改Mysql数据库名称:

```yml
    environment:
        MYSQL_DATABASE: laradock
    ...
```

把Redis默认端口改为1111:

```yml
    ports:
        - "1111:6379"
    ...
```



## 编辑一个Docker镜像

### 1.找到你想编辑的镜像的`Dockerfile`文件

例如编辑`mysql`的`mysql/Dockerfile`文件。

### 2.以你想要的方式编辑文件

### 3.重新构建这个容器

```bash
docker-compose build mysql
```
更多关于容器构建的信息请点击[这里](#构建或重构容器)。



## 构建或重构容器

如果你对`Dockerfile`进行了任何修改，请确保你运行这个命令，这些修改才能生效。

```bash
docker-compose build
```

或者，你可以指定要重建的容器（而不是重建所有容器）:

```bash
docker-compose build {container-name}
```

如果你想全部重建，你可以使用`--no-cache`

```
docker-compose build --no-cache {container-name}
```


## 添加更多软件的Docker镜像

要添加一个镜像(软件), 只需要编辑`docker-compose.yml`并添加你的容器信息，要做到这一点你需要熟悉[docker compose文件的语法](https://docs.docker.com/compose/compose-file/).


## 查看日志文件

NGINX 日志文件存储在`logs/nginx`目录中

但是要查看其他容器（Mysql，PHP-FPM,...）的日志，可以运行以下命令：

```bash
docker-compose logs {container-name}
```

```bash
docker-compose logs -f {container-name}
```

更多[选项](https://docs.docker.com/compose/reference/logs/)




## 安装PHP扩展

在安装PHP扩展之前，你必须决定你需要`FPM`还是`CLI`，因为每个都在不同的容器上，如果你两个都需要，你必须编辑这两个容器。

PHP-FPM扩展应该安装在`php-fpm/Dockerfile-XX`. *(用您的默认PHP版本号替换XX)*.

PHP-CLI扩展应该安装在`workspace/Dockerfile`.




## 修改PHP-FPM版本

默认运行的是**PHP-FPM 7.0**

> PHP-FPM负责为您的应用程序代码提供服务,如果您计划在不同的PHP-FPM版本上运行您的应用程序，则不必更改PHP-CLI的版本。


### A.从PHP `7.0`切换到PHP `5.6`

1. 打开`docker-compose.yml`

2. 在PHP容器部分搜索`Dockerfile-70`

3. 通过使用 `Dockerfile-56`替换`Dockerfile-70`来改变版本号, 就像这样:
```yml
    php-fpm:
        build:
            context: ./php-fpm
            dockerfile: Dockerfile-56
    ...
```
4. 最后重新构建容器

```bash
docker-compose build php-fpm
```

> 关于更多PHP基础镜像的信息，请访问[PHP官方Docker镜像](https://hub.docker.com/_/php/).


### B.切换PHP `7.0` 或 `5.6` 为PHP `5.5`

我们不在原生支持PHP5.5，单您可以通过几个步骤获得它：
1. 克隆 `https://github.com/laradock/php-fpm`
2. 重命名 `Dockerfile-56` to `Dockerfile-55`
3. 编辑`FROM php:5.6-fpm` to `FROM php:5.5-fpm`
4. 构建一个`Dockerfile-55`的镜像
5. 打开`docker-compose.yml`文件
6. 将 `php-fpm`指向你的`Dockerfile-55`文件




## 修改PHP-CLI版本

默认情况下使用的是**PHP-CLI 7.0**

> 注意: 编辑PHP-CLI版本并不是非常重要。PHP-CLI仅用于Artisan Commands＆Composer。它不提供应用程序代码，这是PHP-FPM的工作。

PHP-CLI安装在Workspace容器中,要更改PHP-CLI版本，您需要编辑`workspace/Dockerfile`

现在，您必须手动编辑`Dockerfile`或创建一个新的PHP-FPM


## 安装xDebug

1. 首先在Workspace容器中和PHP-FPM容器中安装`xDebug`
    - 打开`docker-compose.yml`文件
    - 在Workspace容器中搜索`INSTALL_XDEBUG`参数
    - 将它设置为true
    - 在PHP-FPM容器中搜索`INSTALL_XDEBUG`参数
    - 将它设置为true

它将是这样的：
```yml
    workspace:
        build:
            context: ./workspace
            args:
                - INSTALL_XDEBUG=true
    ...
    php-fpm:
        build:
            context: ./php-fpm
            args:
                - INSTALL_XDEBUG=true
    ...
```
2. 打开`laradock/workspace/xdebug.ini`和`laradock/php-fpm/xdebug.ini`并至少启用以下配置：

    ```
    xdebug.remote_autostart=1
    xdebug.remote_enable=1
    xdebug.remote_connect_back=1
    ```

3. 重新构建容器`docker-compose build workspace php-fpm`

有关如何使用IDE配置xDebug工作，请查看此[仓库](https://github.com/LarryEitel/laravel-laradock-phpstorm)，如果你在Linux下使用Phpstorm，请继续阅读下一节。



## 在Linux上为PhpStorm设置远程调试

> 确保你已经按照上面的步骤[安装XDebug](http://laradock.io/documentation/#install-xdebug).

> 确保Xdebug接收连接并监听端口9000(应该是默认配置)


![Debug Configuration](/images/photos/PHPStorm/linux/configuration/debugConfiguration.png "Debug Configuration").

> 创建一个名为`laradock`的服务器（与环境中的PHP_IDE_CONFIG键匹配），并确保将项目根路径与服务器正确映射。

![Server Configuration](/images/photos/PHPStorm/linux/configuration/serverConfiguration.png "Server Configuration").

> 开始监听调试连接，放置一个断点，干得漂亮！


## 开始或停止xDebug

通过安装xDebug,你可以在默认情况下启用时运行。

要控制xDeubg（在php-fpm容器中）的行为，可以在laradock根文件夹运行以下命令（在运行docker-compose提示符的同一地方）

- 从默认运行中停止xDebug：`.php-fpm/xdebug stop`
- 打开xDebug：`.php-fpm/xdebug start`
- 查看状态: `.php-fpm/xdebug status`.

注意: 如果`.php-fpm/xdebug`不执行并给出`Permission Denied`错误，问题可能是`xdebug`文件没有执行权限，可以运行`chmod`命令来设置所需访问权限。


## 安装PHP部署工具Deployer

1. 打开`docker-compose.yml`文件
2. 在Workspace容器中搜索参数`INSTALL_DEPLOYER`
3. 将其设置为true
    应该是这样的:
    ```yml
        workspace:
            build:
                context: ./workspace
                args:
                    - INSTALL_DEPLOYER=true
        ...
    ```
4.重新构建这个容器`docker-compose build workspace`

[**Deployer文档在这**](https://deployer.org/docs)



## 准备生产环境的Laradock

建议为生产环境创建一个自定义的`docker-compose.yml`文件，因此Laradock附带的`production-docker-compose.yml`文件应当包含您计划在生产中运行的容器(使用示例： `docker-compose -f production-docker-compose.yml up -d nginx mysql redis ...`).

注意: 数据库(MySQL/MariaDB/...) 端口不应该在生产环境中转发, 口不应该在生产中转发，因为Docker会自动发布主机上的端口，这是非常不安全的，除非特别告知不要。所以一定要删除这些行：
```
ports:
    - "3306:3306"
```

要详细了解Docker如何发布端口，请阅读[关于此主题的优秀文章](https://fralef.me/docker-and-iptables.html).



## 在Digital-Ocean上安装Laravel和Docker

[这是全部指南](https://github.com/laradock/laradock/blob/master/_guides/digital_ocean.md)



## 使用Jenkins

1. 启动容器`docker-compose up -d jenkins`，进入容器方式`docker-compose exec jenkins bash`
2. 浏览器打开`http://localhost:8090/`（如果你没用修改默认端口映射）
3. 从web应用程序进行身份验证
    - 默认用户名是`admin`
    - 默认密码是`docker-compose exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword`
    (以root方式进入容器`docker-compose exec --user root jenkins bash`)
4. 安装一些插件
5. 创建你的第一个admin用户或继续组委管理员

注意: 要添加用户，请进入 `http://localhost:8090/securityRealm/addUser` 并访问`http://localhost:8090/restart`进行重启

你可能想更改默认安全配置，请进入`http://localhost:8090/configureSecurity/`在授权下选择“Anyone can do anything（任何人都可以执行任何操作）”或Project-based Matrix Authorization Strategy（基于项目的矩阵授权策略”或其他任何操作）。


## 在Docker容器中安装Laravel

1. 首先你需要进入Workspace容器
2. 安装Laravel
    > 使用composer的例子

    ```bash
    composer create-project laravel/laravel my-cool-app "5.2.*"
    ```
    > 我们推荐使用`composer create-project`代替Laravel安装器，来安装Laravel
    有关Laravel安装的更多信息，请点击[此处](https://laravel.com/docs/master#installing-laravel)
3. 编辑`docker-compose.yml`文件以映射新的应用程序路径
    
    默认情况下，Laradock假定Laravel应用程序位于laradock文件夹的父目录中。
    由于新的Laravel应用位于`my-cool-app`文件夹中，因此我们需要使用`../my-cool-app/:/var/www`替换`../:/var/www`，如下所示：
    ```yaml
        application:
            image: tianon/true
            volumes:
                - ../my-cool-app/:/var/www
        ...
    ```
4. 进入该文件并开始工作
    ```bash
    cd my-cool-app
    ```
5. 返回到Laradock安装步骤，了解如何编辑`.env`文件


## 运行Artisan命令

你可以在workspace容器中运行artisan命令和许多其他终端命令。

1. 确保你的workspace容器是出在运行中的。
    ```bash
    docker-compose up -d workspace // ..and all your other containers
    ```
2. 找到workspace容器名称：
    ```bash
    docker-compose ps
    ```
3. 进入workspace容器
    ```bash
    docker-compose exec workspace bash
    ```
    添加`--user=laradock`（例如`docker-compose exec --user=laradock workspace bash`）以创建你的主用户
4. 运行任何你想要的:)
    ```bash
    php artisan
    ```
    ```bash
    Composer update
    ```
    ```bash
    phpunit
    ```

## 运行Laravel队列工作者

1. 首先添加`php-worker`容器，这和PHP-FPM容器类似。
    - 打开`docker-compose.yml`文件
    - 只需在PHP-FPM容器下复制粘贴本节即可添加一个新的服务容器
    ```yaml
        php-worker:
        build:
            context: ./php-worker
            dockerfile: "Dockerfile-${PHP_VERSION}" #Dockerfile-71 or #Dockerfile-70 available
            args:
            - INSTALL_PGSQL=${PHP_WORKER_INSTALL_PGSQL} #Optionally install PGSQL PHP drivers
        volumes_from:
            - applications
        depends_on:
            - workspace
        extra_hosts:
            - "dockerhost:${DOCKER_HOST_IP}"
        networks:
            - backend
    ```
2. 开始一切
    ```bash
    docker-compose up -d php-worker
    ```

## 使用Redis

1. 首先确保你的`redis`容器使用`docker-compose up`启动
    ```bash
    docker-compose up -d redis
    ```
    > 要执行redis命令，首先进入redis容器`docker-compose exec redis bash`，然后进入`redis-cli`
2. 打开你的Laravel的`.env`文件并设置`REDIS_HOST` 为 `redis
    ```env
    REDIS_HOST=redis
    ```
    如果你正在使用Laravel，并且你么有在你的`.env`文件中找到`REDIS_HOST`变量，转到数据库配置文件`config/database.php`并用`redis`替换默认`127.0.0.1`，Redis如下所示:
    ```php
    'redis' => [
        'cluster' => false,
        'default' => [
            'host'     => 'redis',
            'port'     => 6379,
            'database' => 0,
        ],
    ],
    ```
3. 启用Redis缓存或用于会话管理。还要从`.env`文件中设置`CACHE_DRIVER`和`ESSION_DRIVER`为redis，来替换默认的`file`。
    ```env
    CACHE_DRIVER=redis
    SESSION_DRIVER=redis
    ```
4. 最终确保你通过composer安装了`predis/predis`包 `(~1.0)`
    ```bash
    composer require predis/predis:^1.0
    ```
5. 您可以使用以下代码从Laravel手动测试它
    ```php
    \Cache::store('redis')->put('Laradock', 'Awesome', 10);
    ```

## 使用Mongo

1. 首先在Workspace和PHP-FPM容器中安装`mongo`
    - 打开`docker-compose.yml`文件
    - 在Workspace容器中搜索参数`INSTALL_MONGO`
    - 将它设置为`true`
    - 在PHP-FPM容器中搜索参数`INSTALL_MONGO`
    - 将它设置为`true`
    应该是这样的：
    ```yml
        workspace:
            build:
                context: ./workspace
                args:
                    - INSTALL_MONGO=true
        ...
        php-fpm:
            build:
                context: ./php-fpm
                args:
                    - INSTALL_MONGO=true
        ...
    ```
2. 重新构建容器`docker-compose build workspace php-fpm`
3. 使用`docker-compose up`命令运行 MongoDB容器
    ```bash
    docker-compose up -d mongo
    ```
4. 将MongoDB配置添加到`config/database.php`配置文件中
    ```php
    'connections' => [

        'mongodb' => [
            'driver'   => 'mongodb',
            'host'     => env('DB_HOST', 'localhost'),
            'port'     => env('DB_PORT', 27017),
            'database' => env('DB_DATABASE', 'database'),
            'username' => '',
            'password' => '',
            'options'  => [
                'database' => '',
            ]
        ],

        // ...

    ],
    ```
5. 打开你的Laravel`.env`文件并更新以下变量
    - 设置`DB_HOST`为你的`mongo`.
    - 设置`DB_PORT`为`27017`.
    - 设置`DB_DATABASE`为`database`.
6. 最后确保你已经通过Composer安装了`jenssegers/mongodb`软件包，并且添加了它的服务提供者

    ```bash
    composer require jenssegers/mongodb
    ```
    更多细节请点击[这里](https://github.com/jenssegers/laravel-mongodb#installation).
7. 测试
 - 先让你的Model继承自Mongo Eloquent模型，点击[文档](https://github.com/jenssegers/laravel-mongodb#eloquent)
 - 进入Workspace容器
 - 迁移数据库`php artisan migrate`


## 使用PhpMyAdmin

1. 使用`docker-compose up`命令运行phpMyAdmin容器
    ```bash
    # use with mysql
    docker-compose up -d mysql phpmyadmin

    # use with mariadb
    docker-compose up -d mariadb phpmyadmin
    ```
    *注意: 要使用MariaDB, 打开 `.env`并设置`PMA_DB_ENGINE=mysql`为 `PMA_DB_ENGINE=mariadb`*
2. 打开浏览器并通过端口**8080**访问本地主机：`http://localhost:8080`


## 使用Adminer

1. 使用`docker-compose up`命令运行Adminer容器，例子：
    ```bash
    docker-compose up -d adminer
    ```
2. 打开浏览器并通过端口**8080**访问本地主机：`http://localhost:8080`
    **注意:** 在撰写本文时，我们已将Adminer锁定为4.3.0版本，其中包含阻止PostgreSQL的[主要错误](https://sourceforge.net/p/adminer/bugs-and-features/548/)如果该错误得到解决（或者您未使用PostgreSQL），请随时设置Adminer到[the Dockerfile](https://github.com/laradock/laradock/blob/master/adminer/Dockerfile#L1)中的最新版本


## 使用PgAdmin

1. 使用`docker-compose up`命令运行pgAdmin容器，例子：
    ```bash
    docker-compose up -d postgres pgadmin
    ```
2. 打开你的浏览器并通过端口**5050**访问本地主机：`http://localhost:5050`


## 使用Beanstalkd

1. 运行Beanstalkd容器
    ```bash
    docker-compose up -d beanstalkd
    ```
2. 通过编辑`config/queue.php`配置文件来配置Laravel以连接到该容器
    - 首先设置`beanstalkd`为默认队列驱动
    - 设置队列主机为beanstalkd：`QUEUE_HOST=beanstalkd`
    *beanstalkd现在可用于默认端口`11300`*
3. 需要使用composer的依赖包[pda/pheanstalk](https://github.com/pda/pheanstalk)
    或者，您可以使用Beanstalkd控制台容器从Web界面管理您的队列
    - 运行Beanstalkd控制台容器
    ```bash
    docker-compose up -d beanstalkd-console
    ```
    - 打开你的浏览器，访问`http://localhost:2080/`
    - 添加服务器
    ```
    Host: beanstalkd
    Port: 11300
    ```
4. 完成


## 使用ElasticSearch

1. 使用`docker-compose up`命令运行ElasticSearch容器
    ```bash
    docker-compose up -d elasticsearch
    ```
2. 打开浏览器并通过端口**9200**访问本地主机：`http://localhost:9200`
> 默认用户是`user` 默认密码是`changeme`

### 安装 ElasticSearch 插件
    - 安装一个ElasticSearch插件
    ```bash
    docker-compose exec elasticsearch /usr/share/elasticsearch/bin/plugin install {plugin-name}
    ```
    - 重启elasticsearch容器
    ```bash
    docker-compose restart elasticsearch
    ```

## 使用Selenium

- 使用`docker-compose up`命令运行Selenium容器
    ```bash
    docker-compose up -d selenium
    ```
- 打开浏览器并通过以下URL 访问端口**4444**上的本地主机：`http://localhost:4444/wd/hub`


## 使用RethinkDB

RethinkDB是一个开源的实时Web数据库[RethinkDB](https://rethinkdb.com/)。

[Laravel RethinkDB](https://github.com/duxet/laravel-rethinkdb)扩展包正在开发中，并且发布了Laravel5.2的released版本

1. 使用`docker-compose up`命令运行RethinkDB容器
    ```bash
    docker-compose up -d rethinkdb
    ```
2. 通过RethinkDB管理控制台[http://localhost:8090/#tables](http://localhost:8090/#tables)来创建一个叫做`database`的数据库
3. 将RethinkDB配置添加到`config/database.php`配置文件中
    ```php
    'connections' => [

        'rethinkdb' => [
            'name'      => 'rethinkdb',
            'driver'    => 'rethinkdb',
            'host'      => env('DB_HOST', 'rethinkdb'),
            'port'      => env('DB_PORT', 28015),
            'database'  => env('DB_DATABASE', 'test'),
        ]

        // ...

    ],
    ```
4. 打开你的laravel的`.env`文件，并更新以下变量：
    - 设置`DB_CONNECTION`为`rethinkdb`.
    - 设置`DB_HOST`为`rethinkdb`.
    - 设置`DB_PORT`为`28015`.
    - 设置`DB_DATABASE`为`database`.


## 使用Minio

1. 配置Minio
    - 在Workspace容器中，修改`INSTALL_MC`为true以获取客户端。
    - 如果您希望设置正确的密钥，设置`MINIO_ACCESS_KEY`和`MINIO_ACCESS_SECRET`
2. 使用`docker-compose up`命令进入Minio容器，例子：
    ```bash
    docker-compose up -d minio
    ```
3. 打开浏览器并通过以下URL，访问**9000**端口上的localhost：`http://localhost:9000`
4. 通过webui或使用mc客户端创建一个bucket
    ```bash
    mc mb minio/bucket
    ```
5. 在配置其他客户端时使用以下详细信息
    ```
    S3_HOST=http://minio
    S3_KEY=access
    S3_SECRET=secretkey
    S3_REGION=us-east-1
    S3_BUCKET=bucket
    ```



## 使用AWS

1. 配置AWS：
    -  确保将您的SSH密钥添加到`aws/ssh_keys`文件夹中
2. 使用`docker-compose up`命令运行Aws容器，栗子：
    ```bash
    docker-compose up -d aws
    ```
3. 使用`docker-compose exec aws bash`命令访问aws容器
4. 开始在容器内使用eb cli，首先通过执行`eb init`来启动你的项目。阅读[aws eb cli](http://docs.aws.amazon.com/elasticbeanstalk/latest/dg/eb-cli3-configuration.html)文档了解更多详情。


## 使用Grafana

1. 配置Grafana：如果你愿意，你可以使用`GRAFANA_PORT`更改端口，默认端口是3000
2. 使用`docker-compose up`命令运行Grafana容器
    ```bash
    docker-compose up -d grafana
    ```
3. 打开浏览器并通过以下URL访问端口**3000**上的本地主机：`http://localhost:3000`
4. 使用凭证登录User=`admin`,Passwort=`admin`。如果需要，请更改Web界面中的密码


## 安装CodeIgniter

要在Laradock上安装CodeIgniter 3，您只需执行以下简单步骤：
1. 打开`docker-compose.yml`文件
2. 修改`CODEIGNITER=false`为`CODEIGNITER=true`
3. 重新构建你的PHP-FPM容器`docker-compose build php-fpm`


## 安装Symfony

1. 打开`.env`文件，设置`WORKSPACE_INSTALL_SYMFONY`为`true`。
2. 执行完上面的命令，运行`docker-compose build workspace`
3. NGINX站点包含Symfony项目的默认配置文件`symfony.conf.example`，因此编辑它并确保root它指向您的项目web目录。
4. 如果容器已经运行，在上述步骤执行之前先运行`docker-compose restart`
5. 访问`symfony.test`


## 更改时区

要更改`workspace`容器中的时区，请将Docker compose文件中的`TZ`构建参数修改为[TZ 数据库](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones)中的一个
举个栗子, 如果你想设置时区为`New York`:
    ```yml
        workspace:
            build:
                context: ./workspace
                args:
                    - TZ=America/New_York
        ...
    ```
我们还建议[在laravel项目中设置时区](http://www.camroncade.com/managing-timezones-with-laravel/).



## 添加定时任务

你可以在`workspace/crontab/root`中添加定时任务（cron jobs），放在`php artisan`这一行后面
    ```
    * * * * * php /var/www/artisan schedule:run >> /dev/null 2>&1

    # Custom cron
    * * * * * root echo "Every Minute" > /var/log/cron.log 2>&1
    ```
如果你不想使用默认时区UTC，请[更改时区](#更改时区)


## 通过ssh访问工作区

您可以在workspace通过`localhost:2222`设置`INSTALL_WORKSPACE_SSH`构建参数来访问容器`true`。
要更改ssh的默认转发端口
    ```yml
        workspace:
            ports:
                - "2222:22" # Edit this line
        ...
    ```


## 更改MySQL版本

默认情况下运行的Mysql版本为8.0

Mysql8.0是一个开发版本，您可能更愿意使用最新的稳定版本，或者更旧的版本。如果你愿意，你可以改变使用的MySQL镜像。

打开你的.env文件，设置`MYSQL_VERSION`变量为你想要安装的版本。
    ```
    MYSQL_VERSION=5.7
    ```
可用版本为：5.5,5.6,5.7,8.0或最新版本。有关更多信息，请参阅https://store.docker.com/images/mysql 



## 通过主机访问Mysql

你可以通过确保这些线被添加到转发的MySQL/MariaDB的端口
你可以转发MySQL/MariaDB端口到你的宿主机上通过确保这些关系被添加到`docker-compose.yml`或[environment specific Compose](https://docs.docker.com/compose/extends/)文件的`mysql`或`mariadb`部分，或在你的

```
ports:
    - "3306:3306"
```


## 使用root访问Mysql

默认的mysql用户的用户名和密码是root和root
1. 进入mysql容器`docker-compose exec mysql bash`
2. 使用root进入mysql`mysql -uroot -proot`，使用费root用户进入mysql`mysql -uhomestead -psecret`
3. 查看所有用户`SELECT User FROM mysql.user;`
4. 运行一些命令`show databases`, `show tables`, `select * from.....`.


## 创建多个MySQL数据库

Create `createdb.sql` from `mysql/docker-entrypoint-initdb.d/createdb.sql.example` in `mysql/docker-entrypoint-initdb.d/*` and add your SQL syntax as follow:

```sql
CREATE DATABASE IF NOT EXISTS `your_db_1` COLLATE 'utf8_general_ci' ;
GRANT ALL ON `your_db_1`.* TO 'mysql_user'@'%' ;
```


## 更改MySQL端口

修改`mysql/my.cnf` 文件并设置你的端口号, `1234` 作为示例
```
[mysqld]
port=1234
```

如果你需要从主机访问mysql，请不要忘记更改docker-compose配置问价中的内部转发端口： (`"3306:3306"` -> `"3306:1234"`)


## 使用自定义域名代替Docker的ip

假设你的自定义域名是`laravel.test`

1. 打开你的`/etc/hosts`文件，并将你的宿主机地址映射`127.0.0.1`到域名`laravel.test`，方法是添加一下内容：
    ```bash
    127.0.0.1    laravel.test
    ```
2. 打开你的浏览器并访问`{http://laravel.test}`
或者，你可以在NGINX配置文件中定义服务器名称，如下所示：
    ```conf
    server_name laravel.test;
    ```


## 启用全局composer构建安装

在构建容器的过程中启用全局 Composer Install可以让你在完成构建后安装和使用容器中的composer的需求。

1. 打开`docker-compose.yml`文件
2. 在Workspace容器中搜索`COMPOSER_GLOBAL_INSTALL`参数并设置为`true`
    她应该是在这样的:
    ```yml
        workspace:
            build:
                context: ./workspace
                args:
                    - COMPOSER_GLOBAL_INSTALL=true
        ...
    ```
3. 现在添加你的依赖关系到`workspace/composer.json`
4. 重新构建Workspace容器`docker-compose build workspace`



## 安装Prestissimo

[Prestissimo](https://github.com/hirak/prestissimo)是composer的一个插件，它可以实现并行安装功能。

1. 启用全局composer构建安装
2. 在Composer中添加prestissimo作为需要
    - 打开`workspace/composer.json`文件
    - 添加`"hirak/prestissimo": "^0.3"`作为需要
    - 重新构建Workspace容器`docker-compose build workspace`


## 安装Node和NVM

在Workspace容器中安装NVM和NodeJs
1. 打开`docker-compose.yml`文件
2. 在Workspace容器中搜索`INSTALL_NODE`参数并将其设置为`true`
    - 它应该是这样的
    ```yml
        workspace:
            build:
                context: ./workspace
                args:
                    - INSTALL_NODE=true
        ...
    ```
3. 重新构建容器`docker-compose build workspace`



## 安装Node和YARN

Yarn是新的JavaScript的包管理器，他比npm快，你可以在[here](http://yarnpkg.com/en/compare)找到他，来将NodeJS和Yarn安装在Workspace容器中：
1. 打开`docker-compose.yml`文件
2. 在Workspace容器中搜索`INSTALL_NODE` 和 `INSTALL_YARN`参数，并将其设置为`true`
    它应该是这样的：
    ```yml
        workspace:
            build:
                context: ./workspace
                args:
                    - INSTALL_NODE=true
                    - INSTALL_YARN=true
        ...
    ```
3. 重新构建容器`docker-compose build workspace`



## 安装Linuxbrew

Linuxbrew是Linux的包管理器，它是MacOSS的Homebrew的Linux版本，可以在[这里](http://linuxbrew.sh)找到它。将Linuxbrew安装在Workspace中：

1. 打开`docker-compose.yml`文件
2. 在Workspace容器中查找`INSTALL_LINUXBREW`参数，并将其设置为`true`
    它应该还是这样的
    ```yml
        workspace:
            build:
                context: ./workspace
                args:
                    - INSTALL_LINUXBREW=true
        ...
    ```
3. 重新构建容器`docker-compose build workspace`


## 常见的Terminal别名
当你启动Docker容器时，Laradock将会赋值位于`laradock/workspace`目录中的`aliases.sh`文件，并将源文件添加到容器的`~/.bashrc`文件中。

你可以任意修改`aliases.sh`为你认为合适的，可以个努努你自己的需要添加你自己的别名（或宏函数），


## 安装Aerospike扩展

1. 首先在Workspace和PHP-FPM容器中安装`aerospike`
    - 打开`docker-compose.yml`文件
    - 在Workspace容器中查找`INSTALL_AEROSPIKE`参数
    - 将它设置为`true`
    它应该是这样的:
    ```yml
        workspace:
            build:
                context: ./workspace
                args:
                    - INSTALL_AEROSPIKE=true
        ...
        php-fpm:
            build:
                context: ./php-fpm
                args:
                    - INSTALL_AEROSPIKE=true
        ...
    ```
2. 重新构建容器`docker-compose build workspace php-fpm`


## 安装Laravel-Envoy

1. 打开`docker-compose.yml`文件
2. 在Workspace容器中搜索`INSTALL_LARAVEL_ENVOY`参数
3. 将它设置为true
    它应该是这样的：
    ```yml
        workspace:
            build:
                context: ./workspace
                args:
                    - INSTALL_LARAVEL_ENVOY=true
        ...
    ```
4. 重新构建容器`docker-compose build workspace`

[**Laravel Envoy 文档在这**](https://laravel.com/docs/5.3/envoy)



## PHPStorm调试指南

远程调试Laravel web和phpunit测试

[**Debugging 指南在这**](https://github.com/laradock/laradock/blob/master/_guides/phpstorm.md)



## 跟踪你的laradock变化

1. Fork the Laradock repository.
2. 将该fork作为子模块
3. 将所有更改提交给你的fork分支
4. 时常从主repository中拉取新的更新


## 升级Laradock

Moving from Docker Toolbox (VirtualBox) to Docker Native (for Mac/Windows). Requires upgrading Laradock from v3.* to v4.*:

1. 停止所有Docker虚拟机`docker-machine stop {default}`
2. 安装 Docker [Mac](https://docs.docker.com/docker-for-mac/) or [Windows](https://docs.docker.com/docker-for-windows/).
3. 将Laradock更新到`v4.*.*` (`git pull origin master`)
4. 根据你的需要使用Laradock: `docker-compose up -d nginx mysql`.

**Note:** 如果你在执行上面的最后一步时遇到任何问题: 重新构建你的容器！
`docker-compose build --no-cache`
"警告：容器数据可能会丢失！"



## 在MacOS上提高速度

> 译者注：关于如何在MacOS上提高执行速度的详细指南请参考[官方英文文档](http://laradock.io/documentation/#improve-speed-on-macos),该部分文档未做翻译。

在撰写本文档时，Docker在Mac上[执行速度很慢](https://github.com/docker/for-mac/issues/77),特别是对于大项目这个可能是一个问题，这个问题早于[2016年3月](https://forums.docker.com/t/file-access-in-mounted-volumes-extremely-slow-cpu-bound/8076)。
因为这是一个长期存在的问题，我们将其纳入文档。

因此，与Linux相比，由于osxfs将代码共享到Docker容器的性能非常差，可能有一些解决方法：

> 译者注：详细解决方法请参考[官方英文文档](http://laradock.io/documentation/#improve-speed-on-macos),该部分文档未做翻译。


## 常见问题

*以下列出了您可能遇到的常见问题以及可能的解决方案*


### 看到一个空白页面，而不是Laravel的`欢迎`页面

从Laravel根目录运行以下命令：
```bash
sudo chmod -R 777 storage bootstrap/cache
```


### 看到"Welcome to nginx"而不是Laravel应用程序

在浏览器中使用`http://127.0.0.1`代替`http://localhost`



### 看到一条错误消息包含`address already in use` 或者 `port is already allocated`

确保您尝试运行的服务的端口（22,80,443,3306等）未被主机上的其他程序使用，例如内置`apache`/`httpd`服务或其他开发工具。



### 在Windows上收到Nginx错误404 Not Found

1. 转到Windows机器上的docker设置
2. 点击`Shared Drives`选项卡并检查包含项目文件的驱动器
3. 输入你的Windows用户名和密码
4. 进入`reset`选项卡，点击重新启动Docker



### 我的服服务中的时间和当前时间不符

1. 确保你已经[修改时区时区](#修改时区)
2. 停止并重新构建容器(`docker-compose up -d --build <services>`)


### Mysql拒绝连接

因为你的Laravel应用并未运行在容器的本机IP(127.0.0.1)上，有时会发生这个问题，可以通过以下步骤来解决：

- 选项A
    1. 通过在你应用程序中的任意位置die dump`Request::ip()`来检查正在运行的Laravel应用的IP，显示的结果是你的Laravel容器的IP。
    2. 使用上一步你获得的IP更改`env`上的`DB_HOST`的变量。
- 选项B
    1. 将`DB_HOST`的值修改为Mysql容器的名字，当前Laradock docker-compose的名称是`mysql`

### I get stuck when building nginx on `fetch http://mirrors.aliyun.com/alpine/v3.5/main/x86_64/APKINDEX.tar.gz`

As stated on [#749](https://github.com/laradock/laradock/issues/749#issuecomment-293296687), removing the line `RUN sed -i 's/dl-cdn.alpinelinux.org/mirrors.aliyun.com/' /etc/apk/repositories` from `nginx/Dockerfile` solves the problem.		
