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

1) Boot the container `docker-compose up -d jenkins`. To enter the container type `docker-compose exec jenkins bash`.

2) Go to `http://localhost:8090/` (if you didn't chanhed your default port mapping) 

3) Authenticate from the web app.

- Default username is `admin`.
- Default password is `docker-compose exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword`. 

(To enter container as root type `docker-compose exec --user root jenkins bash`).

4) Install some plugins.

5) Create your first Admin user, or continue as Admin.

Note: to add user go to `http://localhost:8090/securityRealm/addUser` and to restart it from the web app visit `http://localhost:8090/restart`.

You may wanna change the default security configuration, so go to `http://localhost:8090/configureSecurity/` under Authorization and choosing "Anyone can do anything" or "Project-based Matrix Authorization Strategy" or anything else.



## 在Docker容器中安装Laravel

1 - First you need to enter the Workspace Container.

2 - Install Laravel.

Example using Composer

```bash
composer create-project laravel/laravel my-cool-app "5.2.*"
```

> We recommend using `composer create-project` instead of the Laravel installer, to install Laravel.

For more about the Laravel installation click [here](https://laravel.com/docs/master#installing-laravel).


3 - Edit `docker-compose.yml` to Map the new application path:

By default, Laradock assumes the Laravel application is living in the parent directory of the laradock folder.

Since the new Laravel application is in the `my-cool-app` folder, we need to replace `../:/var/www` with `../my-cool-app/:/var/www`, as follow:

```yaml
    application:
		 image: tianon/true
        volumes:
            - ../my-cool-app/:/var/www
    ...
```
4 - Go to that folder and start working..

```bash
cd my-cool-app
```

5 - Go back to the Laradock installation steps to see how to edit the `.env` file.



## 运行Artisan命令

You can run artisan commands and many other Terminal commands from the Workspace container.

1 - Make sure you have the workspace container running.

```bash
docker-compose up -d workspace // ..and all your other containers
```

2 - Find the Workspace container name:

```bash
docker-compose ps
```

3 - Enter the Workspace container:

```bash
docker-compose exec workspace bash
```

Add `--user=laradock` (example `docker-compose exec --user=laradock workspace bash`) to have files created as your host's user.


4 - Run anything you want :)

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

1 - First add `php-worker` container. It will be similar as like PHP-FPM Container.
<br>
a) open the `docker-compose.yml` file
<br>
b) add a new service container by simply copy-paste this section below PHP-FPM container

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
2 - Start everything up

```bash
docker-compose up -d php-worker
```



## 使用Redis

1 - First make sure you run the Redis Container (`redis`) with the `docker-compose up` command.

```bash
docker-compose up -d redis
```

> To execute redis commands, enter the redis container first `docker-compose exec redis bash` then enter the `redis-cli`.

2 - Open your Laravel's `.env` file and set the `REDIS_HOST` to `redis`

```env
REDIS_HOST=redis
```

If you're using Laravel, and you don't find the `REDIS_HOST` variable in your `.env` file. Go to the database configuration file `config/database.php` and replace the default `127.0.0.1` IP with `redis` for Redis like this:

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

3 - To enable Redis Caching and/or for Sessions Management. Also from the `.env` file set `CACHE_DRIVER` and `SESSION_DRIVER` to `redis` instead of the default `file`.

```env
CACHE_DRIVER=redis
SESSION_DRIVER=redis
```

4 - Finally make sure you have the `predis/predis` package `(~1.0)` installed via Composer:

```bash
composer require predis/predis:^1.0
```

5 - You can manually test it from Laravel with this code:

```php
\Cache::store('redis')->put('Laradock', 'Awesome', 10);
```



## 使用Mongo

1 - First install `mongo` in the Workspace and the PHP-FPM Containers:
<br>
a) open the `docker-compose.yml` file
<br>
b) search for the `INSTALL_MONGO` argument under the Workspace Container
<br>
c) set it to `true`
<br>
d) search for the `INSTALL_MONGO` argument under the PHP-FPM Container
<br>
e) set it to `true`

It should be like this:

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

2 - Re-build the containers `docker-compose build workspace php-fpm`



3 - Run the MongoDB Container (`mongo`) with the `docker-compose up` command.

```bash
docker-compose up -d mongo
```


4 - Add the MongoDB configurations to the `config/database.php` configuration file:

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

5 - Open your Laravel's `.env` file and update the following variables:

- set the `DB_HOST` to your `mongo`.
- set the `DB_PORT` to `27017`.
- set the `DB_DATABASE` to `database`.


6 - Finally make sure you have the `jenssegers/mongodb` package installed via Composer and its Service Provider is added.

```bash
composer require jenssegers/mongodb
```
More details about this [here](https://github.com/jenssegers/laravel-mongodb#installation).

7 - Test it:

- First let your Models extend from the Mongo Eloquent Model. Check the [documentation](https://github.com/jenssegers/laravel-mongodb#eloquent).
- Enter the Workspace Container.
- Migrate the Database `php artisan migrate`.



## 使用PhpMyAdmin

1 - Run the phpMyAdmin Container (`phpmyadmin`) with the `docker-compose up` command. Example:

```bash
# use with mysql
docker-compose up -d mysql phpmyadmin

# use with mariadb
docker-compose up -d mariadb phpmyadmin
```

*Note: To use with MariaDB, open `.env` and set `PMA_DB_ENGINE=mysql` to `PMA_DB_ENGINE=mariadb`.*

2 - Open your browser and visit the localhost on port **8080**:  `http://localhost:8080`



## 使用Adminer

1 - Run the Adminer Container (`adminer`) with the `docker-compose up` command. Example:

```bash
docker-compose up -d adminer
```

2 - Open your browser and visit the localhost on port **8080**:  `http://localhost:8080`

**Note:** We've locked Adminer to version 4.3.0 as at the time of writing [it contained a major bug](https://sourceforge.net/p/adminer/bugs-and-features/548/) preventing PostgreSQL users from logging in. If that bug is fixed (or if you're not using PostgreSQL) feel free to set Adminer to the latest version within [the Dockerfile](https://github.com/laradock/laradock/blob/master/adminer/Dockerfile#L1): `FROM adminer:latest`



## 使用PgAdmin

1 - Run the pgAdmin Container (`pgadmin`) with the `docker-compose up` command. Example:

```bash
docker-compose up -d postgres pgadmin
```

2 - Open your browser and visit the localhost on port **5050**:  `http://localhost:5050`



## 使用Beanstalkd

1 - Run the Beanstalkd Container:

```bash
docker-compose up -d beanstalkd
```

2 - Configure Laravel to connect to that container by editing the `config/queue.php` config file.

a. first set `beanstalkd` as default queue driver
b. set the queue host to beanstalkd : `QUEUE_HOST=beanstalkd`

*beanstalkd is now available on default port `11300`.*

3 - Require the dependency package [pda/pheanstalk](https://github.com/pda/pheanstalk) using composer.


Optionally you can use the Beanstalkd Console Container to manage your Queues from a web interface.

1 - Run the Beanstalkd Console Container:

```bash
docker-compose up -d beanstalkd-console
```

2 - Open your browser and visit `http://localhost:2080/`

3 - Add the server

- Host: beanstalkd
- Port: 11300

4 - Done.



## 使用ElasticSearch

1 - Run the ElasticSearch Container (`elasticsearch`) with the `docker-compose up` command:

```bash
docker-compose up -d elasticsearch
```

2 - Open your browser and visit the localhost on port **9200**:  `http://localhost:9200`

> The default username is `user` and the default password is `changeme`.

### Install ElasticSearch Plugin

1 - Install an ElasticSearch plugin.

```bash
docker-compose exec elasticsearch /usr/share/elasticsearch/bin/plugin install {plugin-name}
```

2 - Restart elasticsearch container

```bash
docker-compose restart elasticsearch
```



## 使用Selenium

1 - Run the Selenium Container (`selenium`) with the `docker-compose up` command. Example:

```bash
docker-compose up -d selenium
```

2 - Open your browser and visit the localhost on port **4444** at the following URL:  `http://localhost:4444/wd/hub`



## 使用RethinkDB

The RethinkDB is an open-source Database for Real-time Web ([RethinkDB](https://rethinkdb.com/)).
A package ([Laravel RethinkDB](https://github.com/duxet/laravel-rethinkdb)) is being developed and was released a version for Laravel 5.2 (experimental).

1 - Run the RethinkDB Container (`rethinkdb`) with the `docker-compose up` command.

```bash
docker-compose up -d rethinkdb
```

2 - Access the RethinkDB Administration Console [http://localhost:8090/#tables](http://localhost:8090/#tables) for create a database called `database`.

3 - Add the RethinkDB configurations to the `config/database.php` configuration file:

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

4 - Open your Laravel's `.env` file and update the following variables:

- set the `DB_CONNECTION` to your `rethinkdb`.
- set the `DB_HOST` to `rethinkdb`.
- set the `DB_PORT` to `28015`.
- set the `DB_DATABASE` to `database`.



## 使用Minio

1 - Configure Minio:
  - On the workspace container, change `INSTALL_MC` to true to get the client
  - Set `MINIO_ACCESS_KEY` and `MINIO_ACCESS_SECRET` if you wish to set proper keys

2 - Run the Minio Container (`minio`) with the `docker-compose up` command. Example:

```bash
docker-compose up -d minio
```

3 - Open your browser and visit the localhost on port **9000** at the following URL:  `http://localhost:9000`

4 - Create a bucket either through the webui or using the mc client:
  ```bash
  mc mb minio/bucket
  ```

5 - When configuring your other clients use the following details:
  ```
  S3_HOST=http://minio
  S3_KEY=access
  S3_SECRET=secretkey
  S3_REGION=us-east-1
  S3_BUCKET=bucket
  ```



## 使用AWS

1 - Configure AWS:
  - make sure to add your SSH keys in aws/ssh_keys folder

2 - Run the Aws Container (`aws`) with the `docker-compose up` command. Example:

```bash
docker-compose up -d aws
```

3 - Access the aws container with `docker-compose exec aws bash`

4 - To start using eb cli inside the container, initiaze your project first by doing 'eb init'. Read the [aws eb cli](http://docs.aws.amazon.com/elasticbeanstalk/latest/dg/eb-cli3-configuration.html) docs for more details.



## 使用Grafana

1 - Configure Grafana: Change Port using `GRAFANA_PORT` if you wish to. Default is port 3000.

2 - Run the Grafana Container (`grafana`) with the `docker-compose up`command:

```bash
docker-compose up -d grafana
```

3 - Open your browser and visit the localhost on port **3000** at the following URL: `http://localhost:3000`

4 - Login using the credentials User = `admin` Passwort = `admin`. Change the password in the webinterface if you want to.



## 安装CodeIgniter

To install CodeIgniter 3 on Laradock all you have to do is the following simple steps:

1 - Open the `docker-compose.yml` file.

2 - Change `CODEIGNITER=false` to `CODEIGNITER=true`.

3 - Re-build your PHP-FPM Container `docker-compose build php-fpm`.


## 安装Symfony

1 - Open the `.env` file and set `WORKSPACE_INSTALL_SYMFONY` to `true`.

2 - Run `docker-compose build workspace`, after the step above.

3 - The NGINX sites include a default config file for your Symfony project `symfony.conf.example`, so edit it and make sure the `root` is pointing to your project `web` directory.

4 - Run `docker-compose restart` if the container was already running, before the step above.

5 - Visit `symfony.test`



## 更改时区

To change the timezone for the `workspace` container, modify the `TZ` build argument in the Docker Compose file to one in the [TZ database](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones).

For example, if I want the timezone to be `New York`:

```yml
    workspace:
        build:
            context: ./workspace
            args:
                - TZ=America/New_York
    ...
```

We also recommend [setting the timezone in Laravel](http://www.camroncade.com/managing-timezones-with-laravel/).



## 添加定时任务

You can add your cron jobs to `workspace/crontab/root` after the `php artisan` line.

```
* * * * * php /var/www/artisan schedule:run >> /dev/null 2>&1

# Custom cron
* * * * * root echo "Every Minute" > /var/log/cron.log 2>&1
```

Make sure you [change the timezone](#Change-the-timezone) if you don't want to use the default (UTC).



## 通过ssh访问工作区

You can access the `workspace` container through `localhost:2222` by setting the `INSTALL_WORKSPACE_SSH` build argument to `true`.

To change the default forwarded port for ssh:

```yml
    workspace:
		ports:
			- "2222:22" # Edit this line
    ...
```



## 更改MySQL版本

By default **MySQL 8.0** is running.

MySQL 8.0 is a development release.  You may prefer to use the latest stable version, or an even older release.  If you wish, you can change the MySQL image that is used.

Open up your .env file and set the `MYSQL_VERSION` variable to the version you would like to install.

```
MYSQL_VERSION=5.7
```

Available versions are: 5.5, 5.6, 5.7, 8.0, or latest.  See https://store.docker.com/images/mysql for more information.



## 通过主机访问Mysql

You can forward the MySQL/MariaDB port to your host by making sure these lines are added to the `mysql` or `mariadb` section of the `docker-compose.yml` or in your [environment specific Compose](https://docs.docker.com/compose/extends/) file.

```
ports:
    - "3306:3306"
```



## 使用root访问Mysql

The default username and password for the root MySQL user are `root` and `root `.

1 - Enter the MySQL container: `docker-compose exec mysql bash`.

2 - Enter mysql: `mysql -uroot -proot` for non root access use `mysql -uhomestead -psecret`.

3 - See all users: `SELECT User FROM mysql.user;`

4 - Run any commands `show databases`, `show tables`, `select * from.....`.




## 创建多个MySQL数据库

Create `createdb.sql` from `mysql/docker-entrypoint-initdb.d/createdb.sql.example` in `mysql/docker-entrypoint-initdb.d/*` and add your SQL syntax as follow:

```sql
CREATE DATABASE IF NOT EXISTS `your_db_1` COLLATE 'utf8_general_ci' ;
GRANT ALL ON `your_db_1`.* TO 'mysql_user'@'%' ;
```


## 更改MySQL端口

Modify the `mysql/my.cnf` file to set your port number, `1234` is used as an example.

```
[mysqld]
port=1234
```

If you need <a href="#MySQL-access-from-host">MySQL access from your host</a>, do not forget to change the internal port number (`"3306:3306"` -> `"3306:1234"`) in the docker-compose configuration file.





## 使用自定义域名代替Docker的ip

Assuming your custom domain is `laravel.test`

1 - Open your `/etc/hosts` file and map your localhost address `127.0.0.1` to the `laravel.test` domain, by adding the following:

```bash
127.0.0.1    laravel.test
```

2 - Open your browser and visit `{http://laravel.test}`


Optionally you can define the server name in the NGINX configuration file, like this:

```conf
server_name laravel.test;
```



## 启用全局composer构建安装

Enabling Global Composer Install during the build for the container allows you to get your composer requirements installed and available in the container after the build is done.

1 - Open the `docker-compose.yml` file

2 - Search for the `COMPOSER_GLOBAL_INSTALL` argument under the Workspace Container and set it to `true`

It should be like this:

```yml
    workspace:
        build:
            context: ./workspace
            args:
                - COMPOSER_GLOBAL_INSTALL=true
    ...
```
3 - Now add your dependencies to `workspace/composer.json`

4 - Re-build the Workspace Container `docker-compose build workspace`




## 安装Prestissimo

[Prestissimo](https://github.com/hirak/prestissimo) is a plugin for composer which enables parallel install functionality.

1 - Enable Running Global Composer Install during the Build:

Click on this [Enable Global Composer Build Install](#Enable-Global-Composer-Build-Install) and do steps 1 and 2 only then continue here.

2 - Add prestissimo as requirement in Composer:

a - Now open the `workspace/composer.json` file

b - Add `"hirak/prestissimo": "^0.3"` as requirement

c - Re-build the Workspace Container `docker-compose build workspace`



## 安装Node和NVM

To install NVM and NodeJS in the Workspace container

1 - Open the `docker-compose.yml` file

2 - Search for the `INSTALL_NODE` argument under the Workspace Container and set it to `true`

It should be like this:

```yml
    workspace:
        build:
            context: ./workspace
            args:
                - INSTALL_NODE=true
    ...
```

3 - Re-build the container `docker-compose build workspace`




## 安装Node和YARN

Yarn is a new package manager for JavaScript. It is so faster than npm, which you can find [here](http://yarnpkg.com/en/compare).To install NodeJS and [Yarn](https://yarnpkg.com/) in the Workspace container:

1 - Open the `docker-compose.yml` file

2 - Search for the `INSTALL_NODE` and `INSTALL_YARN` argument under the Workspace Container and set it to `true`

It should be like this:

```yml
    workspace:
        build:
            context: ./workspace
            args:
                - INSTALL_NODE=true
                - INSTALL_YARN=true
    ...
```

3 - Re-build the container `docker-compose build workspace`




## 安装Linuxbrew

Linuxbrew is a package manager for Linux. It is the Linux version of MacOS Homebrew and can be found [here](http://linuxbrew.sh). To install Linuxbrew in the Workspace container:

1 - Open the `docker-compose.yml` file

2 - Search for the `INSTALL_LINUXBREW` argument under the Workspace Container and set it to `true`

It should be like this:

```yml
    workspace:
        build:
            context: ./workspace
            args:
                - INSTALL_LINUXBREW=true
    ...
```

3 - Re-build the container `docker-compose build workspace`



## 常见的Terminal别名

When you start your docker container, Laradock will copy the `aliases.sh` file located in the `laradock/workspace` directory and add sourcing to the container `~/.bashrc` file.

You are free to modify the `aliases.sh` as you see fit, adding your own aliases (or function macros) to suit your requirements.



## 安装Aerospike扩展

1 - First install `aerospike` in the Workspace and the PHP-FPM Containers:
<br>
a) open the `docker-compose.yml` file
<br>
b) search for the `INSTALL_AEROSPIKE` argument under the Workspace Container
<br>
c) set it to `true`
<br>
d) search for the `INSTALL_AEROSPIKE` argument under the PHP-FPM Container
<br>
e) set it to `true`

It should be like this:

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

2 - Re-build the containers `docker-compose build workspace php-fpm`




## 安装Laravel-Envoy

1 - Open the `docker-compose.yml` file
<br>
2 - Search for the `INSTALL_LARAVEL_ENVOY` argument under the Workspace Container
<br>
3 - Set it to `true`
<br>

It should be like this:

```yml
    workspace:
        build:
            context: ./workspace
            args:
                - INSTALL_LARAVEL_ENVOY=true
    ...
```

4 - Re-build the containers `docker-compose build workspace`

[**Laravel Envoy Documentation Here**](https://laravel.com/docs/5.3/envoy)





## PHPStorm调试指南

Remote debug Laravel web and phpunit tests.

[**Debugging Guide Here**](https://github.com/laradock/laradock/blob/master/_guides/phpstorm.md)




## 跟踪你的laradock变化

1. Fork the Laradock repository.
2. Use that fork as a submodule.
3. Commit all your changes to your fork.
4. Pull new stuff from the main repository from time to time.



## 升级Laradock

Moving from Docker Toolbox (VirtualBox) to Docker Native (for Mac/Windows). Requires upgrading Laradock from v3.* to v4.*:

1. Stop the docker VM `docker-machine stop {default}`
2. Install Docker for [Mac](https://docs.docker.com/docker-for-mac/) or [Windows](https://docs.docker.com/docker-for-windows/).
3. Upgrade Laradock to `v4.*.*` (`git pull origin master`)
4. Use Laradock as you used to do: `docker-compose up -d nginx mysql`.

**Note:** If you face any problem with the last step above: rebuild all your containers
`docker-compose build --no-cache`
"Warning Containers Data might be lost!"



## 在MacOS上提高速度

Docker on the Mac [is slow](https://github.com/docker/for-mac/issues/77), at the time of writing. Especially for larger projects, this can be a problem. The problem is [older than March 2016](https://forums.docker.com/t/file-access-in-mounted-volumes-extremely-slow-cpu-bound/8076) - as it's a such a long-running issue, we're including it in the docs here.

So since sharing code into Docker containers with osxfs have very poor performance compared to Linux. Likely there are some workarounds:



### Workaround A: using dinghy

[Dinghy](https://github.com/codekitchen/dinghy) creates its own VM using docker-machine, it will not modify your existing docker-machine VMs.

Quick Setup giude, (we recommend you check their docs)

1) `brew tap codekitchen/dinghy`

2) `brew install dinghy`

3) `dinghy create --provider virtualbox` (must have virtualbox installed, but they support other providers if you prefer)

4) after the above command is done it will display some env variables, copy them to the bash profile or zsh or.. (this will instruct docker to use the server running inside the VM)

5) `docker-compose up ...`



<br>
<a name="Docker-Sync"></a>
### Workaround B: using d4m-nfs

You can use the d4m-nfs solution in 2 ways, one is using the Laradock built it integration, and the other is using the tool separatly. Below is show case of both methods:


#### B.1: using the built in d4m-nfs integration

In simple terms, docker-sync creates a docker container with a copy of all the application files that can be accessed very quickly from the other containers.
On the other hand, docker-sync runs a process on the host machine that continuously tracks and updates files changes from the host to this intermediate container.

Out of the box, it comes pre-configured for OS X, but using it on Windows is very easy to set-up by modifying the `DOCKER_SYNC_STRATEGY` on the `.env`

##### Usage

Laradock comes with `sync.sh`, an optional bash script, that automates installing, running and stopping docker-sync.  Note that to run the bash script you may need to change the permissions `chmod 755 sync.sh`

1) Configure your Laradock environment as you would normally do and test your application to make sure that your sites are running correctly.

2) Make sure to set `DOCKER_SYNC_STRATEGY` on the `.env`. Read the [syncing strategies](https://github.com/EugenMayer/docker-sync/wiki/8.-Strategies) for details.
```
# osx: 'native_osx' (default)
# windows: 'unison'
# linux: docker-sync not required

DOCKER_SYNC_STRATEGY=native_osx
```

2) Install the docker-sync gem on the host-machine:
```bash
./sync.sh install
```
3) Start docker-sync and the Laradock environment.
Specify the services you want to run, as you would normally do with `docker-compose up`
```bash
./sync.sh up nginx mysql
```
Please note that the first time docker-sync runs, it will copy all the files to the intermediate container and that may take a very long time (15min+).
4) To stop the environment and docker-sync do:
```bash
./sync.sh down
```

##### Setting up Aliases (optional)

You may create bash profile aliases to avoid having to remember and type these commands for everyday development.
Add the following lines to your `~/.bash_profile`:

```bash
alias devup="cd /PATH_TO_LARADOCK/laradock; ./sync.sh up nginx mysql" #add your services
alias devbash="cd /PATH_TO_LARADOCK/laradock; ./sync.sh bash"
alias devdown="cd /PATH_TO_LARADOCK/laradock; ./sync.sh down"
```

Now from any location on your machine, you can simply run `devup`, `devbash` and `devdown`.


##### Additional Commands

Opening bash on the workspace container (to run artisan for example):
 ```bash
 ./sync.sh bash
 ```
Manually triggering the synchronization of the files:
```bash
./sync.sh sync
```
Removing and cleaning up the files and the docker-sync container. Use only if you want to rebuild or remove docker-sync completely. The files on the host will be kept untouched.
```bash
./sync.sh clean
```


##### Additional Notes

- You may run laradock with or without docker-sync at any time using with the same `.env` and `docker-compose.yml`, because the configuration is overridden automatically when docker-sync is used.
- You may inspect the `sync.sh` script to learn each of the commands and even add custom ones.
- If a container cannot access the files on docker-sync, you may need to set a user on the Dockerfile of that container with an id of 1000 (this is the UID that nginx and php-fpm have configured on laradock). Alternatively, you may change the permissions to 777, but this is **not** recommended.

Visit the [docker-sync documentation](https://github.com/EugenMayer/docker-sync/wiki) for more details.








<br>

#### B.2: using the d4m-nfs tool

[D4m-nfs](https://github.com/IFSight/d4m-nfs) automatically mount NFS volume instead of osxfs one.

1) Update the Docker [File Sharing] preferences:

Click on the Docker Icon > Preferences > (remove everything form the list except `/tmp`).

2) Restart Docker.

3) Clone the [d4m-nfs](https://github.com/IFSight/d4m-nfs) repository to your `home` directory.

```bash
git clone https://github.com/IFSight/d4m-nfs ~/d4m-nfs
```

4) Create (or edit) the file `~/d4m-nfs/etc/d4m-nfs-mounts.txt`, and write the follwing configuration in it:

```txt
/Users:/Users
```

5) Create (or edit) the file `/etc/exports`, make sure it exists and is empty. (There may be collisions if you come from Vagrant or if you already executed the `d4m-nfs.sh` script before).


6) Run the `d4m-nfs.sh` script (might need Sudo):

```bash
~/d4m-nfs/d4m-nfs.sh
```

That's it! Run your containers.. Example:

```bash
docker-compose up ...
```

*Note: If you faced any errors, try restarting Docker, and make sure you have no spaces in the `d4m-nfs-mounts.txt` file, and your `/etc/exports` file is clear.*



## 常见问题

*Here's a list of the common problems you might face, and the possible solutions.*


### I see a blank (white) page instead of the Laravel 'Welcome' page!

Run the following command from the Laravel root directory:

```bash
sudo chmod -R 777 storage bootstrap/cache
```


### I see "Welcome to nginx" instead of the Laravel App!

Use `http://127.0.0.1` instead of `http://localhost` in your browser.



### I see an error message containing `address already in use` or `port is already allocated`

Make sure the ports for the services that you are trying to run (22, 80, 443, 3306, etc.) are not being used already by other programs on the host, such as a built in `apache`/`httpd` service or other development tools you have installed.



### I get NGINX error 404 Not Found on Windows.

1. Go to docker Settings on your Windows machine.
2. Click on the `Shared Drives` tab and check the drive that contains your project files.
3. Enter your windows username and password.
4. Go to the `reset` tab and click restart docker.



### The time in my services does not match the current time

1. Make sure you've [changed the timezone](#Change-the-timezone).
2. Stop and rebuild the containers (`docker-compose up -d --build <services>`)



### I get MySQL connection refused

This error sometimes happens because your Laravel application isn't running on the container localhost IP (Which is 127.0.0.1). Steps to fix it:

* Option A
  1. Check your running Laravel application IP by dumping `Request::ip()` variable using `dd(Request::ip())` anywhere on your application. The result is the IP of your Laravel container.
  2. Change the `DB_HOST` variable on env with the IP that you received from previous step.
* Option B
   1. Change the `DB_HOST` value to the same name as the MySQL docker container. The Laradock docker-compose file currently has this as `mysql`

### I get stuck when building nginx on `fetch http://mirrors.aliyun.com/alpine/v3.5/main/x86_64/APKINDEX.tar.gz`

As stated on [#749](https://github.com/laradock/laradock/issues/749#issuecomment-293296687), removing the line `RUN sed -i 's/dl-cdn.alpinelinux.org/mirrors.aliyun.com/' /etc/apk/repositories` from `nginx/Dockerfile` solves the problem.		
