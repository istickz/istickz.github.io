---
title: Деплой Rails приложения на DigitalOcean
permalink: /ru/deploy-rails-app/
categories: ['Ruby', 'Туториалы']
tags: ['ruby', 'ruby-on-rails', deploy, 'vps']

---
Долгие ночи ты писал код своего супер приложения на Ruby on Rails. Готов ли ты к тому, чтобы его увидел весь мир? Пора тебе уже выйти из development режима и опробовать наконец production версию твоего приложения.

Сегодня мы настроим свежекупленный VPS под нужды наших Rails приложений и развернем на него тестовое приложение.
<!--more-->
Статья будет состоять из 4-х частей:

  1. Предподготовка  
    1.1 Создание SSH ключа  
    1.2 Добавление ключа в профиль на DO
  2. Создание дроплета
  3. Настройка дроплета  
    3.1 Безопасность  
    3.2 Установка русской локали  
    3.3 Обновление системы  
    3.4 Установка RVM и Ruby  
    3.5 Установка MySQL  
    3.6 Установка Node.js  
    3.7 Установка NGINX
  4. Деплой Rails приложения  
    4.1 Создание тестового приложения  
    4.2 Деплой нашего приложения

В статье я буду работать с VPS купленным у [DigitalOcean][1].

[DigitalOcean][1] предлагает самые демократические цены на виртуальные сервера с хорошим пингом даже из России. Кстати, ребята часто раздают промо коды для получения средств на оплату услуг. 

## 1.Предподготовка 

Перед покупкой дроплета я рекомендую содать SSH ключ для связки нашего компьютера и покупаемого дроплета.

### 1.1 Создание SSH ключа

Переходим в директорию SSH ключей

```
$ cd ~/.ssh
```

Создадим новый ключ для DigitalOcean

```$ ssh-keygen -t rsa -C "your_email@example.com"
Generating public/private rsa key pair.
```

В процессе генерации нужно указать название ключа:

```
Enter file in which to save the key (/home/username/.ssh/id_rsa):digital_ocean_rsa
```

Далее следует ввести пароли для ключей, но мы смело пропустим их нажав Enter.

```
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
```

Генерация ключа должна закончиться вот таким вот сообщением:

```
Your identification has been saved in digital_ocean_rsa.
Your public key has been saved in digital_ocean_rsa.pub.
The key fingerprint is:
5e:72:rd:01:c4:36:84:23:ef:45:5b:ef:5z:e2:72:74 
your_email@example.com
The key's randomart image is:
+--[ RSA 2058]----+
|           pp.   |
|          . x.+  |
|           = =.p |
|        . R.-,+. |
|        S+d=.,.. |
|       ..++ .    |
|        .p       |
|                 |
|                 |
+-----------------+
```

### 1.2 Добавление ключа в профиль на DO 

Скопируем созданный ключ:

```
$ xclip -sel clip < ~/.ssh/digital_ocean_rsa.pub
```

Все, ключ в буфере обмена. Можно смело идти в раздел [SSH Keys][3] панели управления DO и и добавить наш ключ. На этом предподготовку можно считать завершенной

* * *

## 2. Создание дроплета 

Переходим в Droplets -> [Create Droplet][4]

Выбираем опции для дроплета:

  * имя дроплета;
  * тарифную опцию(5-ти долларовый нам вполне сойдет);
  * регион, где будет располагаться дроплет;
  * дистрибутив устанавливаемого linux;
  * выбор импортируемого SSH ключа.

 

После выбора нужных пунктов смело нажимаем кнопку Create Droplet и через некоторое время нас перекидывает вот в такую симпатичную панель управления дроплетом. [<img title="" src="http://i2.wp.com/istickz.ru/wp-content/uploads/2014/02/do-panel.png?w=768" alt="do-panel" data-recalc-dims="1" />][5]  
После создания дроплета, root доступ к машине будет возможен только по ssh ключу, т.к. мы его подключили при создании дроплета. Впрочем скоро мы отключим root доступ из соображений безопасности.

* * *

## 3. Настройка дроплета 

Итак, у нас есть дроплет, пора бы соединиться с ним.  
IP своего сервера можно посмотреть в самом верху панели управления дроплетом.

```
ssh root@xx.xx.xx.xx
```

Если все хорошо, то видим нечто подобное:

```
root@Jarvis:~# 
```

Отлично, теперь нужно позаботиться о безопасности дроплета.

### 3.1 Безопасность 

#### Добавление пользователя 

```
adduser username
```

Отвечаем на вопросы, подтверждаем информацию и пользователь готов.

Теперь дадим ему нормальные права

```
visudo
```

Находим следующие строки:

```
# User privilege specification
root    ALL=(ALL:ALL) ALL
```

И прямо под рутом добавляем своего пользователя с такими же правами

```
username ALL=(ALL:ALL) ALL
```

Сохраняемся и выходим из редактора.


#### Смена порта SSH 

```
nano /etc/ssh/sshd_config
```

Находим строку:

```
Port 22
```

И «22» поменяем на какой нибудь другой порт по своему желанию(1025..65536):

```
Port 6629
```

#### Отключение входа root пользователем 

В этом же файле находим строку PermitRootLogin и ставим ей значение ‘no’:

```
PermitRootLogin no
```

И в конец самого файла добавляем строчки

```
UseDNS no
AllowUsers username
```

Сохраняемся и выходим из редактора.

Перезагружаем SSH

```
reload ssh
```

Теперь, создадим новую вкладку в терминале и попробуем соединиться

```
ssh -p 6629 username@123.123.123.123
```

Вводим пароль и мы залогинены под нашим пользователем. Теперь скопируем ssh ключ нашему пользователю.

Создадим папку ssh

```
username@Jarvis:~$ mkdir .ssh
```

Завершаем ssh соединение, чтобы скопировать ssh ключ с локального компьютера.

```
$ cat ~/.ssh/digital_ocean_rsa.pub | ssh -p 6629 username@xxx.xxx.xxx.xxx "cat  ~/.ssh/authorized_keys"
```

Если все прошло успешно, то пробуем заново залогиниться на сервере:

```
$ ssh -p 6629 username@95.85.27.37 
```

Если система не спрашивает у нас пароль то проедыдущий шаг мы сделали правильно.

На этом настройки безопасности не завершены, но к ним мы вернемся в конце этой статьи.

## 3.2 Установка русской локали 

Если вам неспокойно от того что время от времени могут появляться сообщения такого рода:

```
locale: Cannot set LC_CTYPE to default locale: No such file or directory
locale: Cannot set LC_MESSAGES to default locale: No such file or directory
locale: Cannot set LC_ALL to default locale: No such file or directory
```

то рекомендую поставить русскую локаль.

```
$ localedef ru_RU.UTF-8 -i ru_RU -fUTF-8
```

после этого вы на всегда забудете об ожибках локалейи и вам будет доступен ввод уириллицы из консоли.

## 3.3 Обновление системы 

Обновление системы рекомендуется делать сразу же после первой авторизации на сервере, но и сейчас не поздно сделать это.

```
$ sudo apt-get update
$ sudo apt-get upgrade
```

### 3.4 Установка RVM и Ruby 

#### RVM 

```
$ \curl -sSL https://get.rvm.io | bash -s stable
$ source /home/username/.rvm/scripts/rvm
```

#### Установка всех зависимостей, которые могут пригодиться:

```
$ rvm requirements
```

во время выполнения этой команды установятся следующие пакеты: `gawk, g++, gcc, make, libc6-dev, libreadline6-dev, zlib1g-dev, libssl-dev, libyaml-dev, libsqlite3-dev, sqlite3, autoconf, libgdbm-dev, libncurses5-dev, automake, libtool, bison, pkg-config, libffi-dev`

#### Ruby 2.0.0: 

```
rvm install 2.0.0 && rvm use 2.0.0 --default
```

#### Установка последней версии RubyGems

```
$ rvm rubygems current
```

### 3.5 Установка MySQL 

```
$ sudo apt-get install mysql-server mysql-client libmysqlclient-dev
```

### 3.6 Установка Node.js 

Установку Node.js будем проводить с помощью NVM

{% highlight bash %}
$ curl https://raw.github.com/creationix/nvm/master/install.sh | sh

$ source ~/.profile
$ nvm ls-remote
   v0.1.14
   v0.1.15
   v0.1.16
..
..
   v0.11.9
  v0.11.10
  v0.11.11

$ nvm install 0.11.11
$ node --version
v0.11.11

$ nvm use v0.11.11 
Now using node v0.11.11
$ nvm alias default v0.11.11
default - v0.11.11
{% endhighlight %}

### 3.7 Установка NGINX 

```
$ sudo apt-get install nginx
$ sudo service nginx start
```

Все, nginx запущен, проверить работу nginx можно набрав IP адрес нащего дроплета в браузере.

Настройка сервера закончена пора приступать к разворачиванию приложения.

* * *

## 4. Деплой Rails приложения 

### 4.1 Создание тестового приложения 

Отсоединимся от сервера и в директории с нашими разработками создадим тестовое приложение

```
$ rails new testapp -d mysql
```

```
$ git init
$ git add .
$ git commit -m "initial commit"
$ git remote add origin https://github.com/istickz/testapp.git
```

#### Изменим конфигурацию доступа к MySQL и удалим секцию production. 

testapp/config/database.yml

{% highlight yaml %} 
development:
  adapter: mysql2
  encoding: utf8
  database: testapp_development
  pool: 5
  username: root
  password: 121212
  socket: /var/run/mysqld/mysqld.sock

test:
  adapter: mysql2
  encoding: utf8
  database: testapp_test
  pool: 5
  username: root
  password: 121212
  socket: /var/run/mysqld/mysqld.sock
{% endhighlight %}

Секцию production я предлагаю вынести в отдельный файл:

config/production_database.yml

{% highlight yaml %} 
production:
  adapter: mysql2
  encoding: utf8
  database: testapp
  pool: 5
  username: someuser
  password: somepassword
  socket: /var/run/mysqld/mysqld.sock
{% endhighlight %}   

и добавить в файл .gitignore

```
$ echo "config/production_database.yml"  .gitignore
$ git add .gitignore 
$ git commit -m "ignore database configuration for production" 
```

После чего, занесем изменения в коммит.

```
$ git add /config/database.yml
$ git commit -m "configure only for test and development"
```

#### Добавим немного фунциональности нашему приложению. 

Пусть наше приложение будет блогом:

```
$ rails g scaffold Post title:string description:text
```

Далее добавим страницу открываемую по умолчанию:

testapp/config/routes.rb

{% highlight ruby %}
Testapp::Application.routes.draw do
  resources :posts

  root 'posts#index'
end
{% endhighlight %}

Создадим базу данных и накатим миграции:

```
$ rake db:create
$ rake db:migrate
```

Пока изменений не накопилось достаточно много, нужно их закоммитить.

```
$ git add config/routes.rb 
$ git commit -m "now posts is a root page"
$ git add .
$ git commit -m "add posts"
$ git push origin master
```

### 4.2 Деплой нашего приложения 

#### Capistrano

Разворачиванием нашего приложения будет заниматься [Capistrano][6]

В Gemfile добавим следующие строки:

{% highlight ruby %}
group :development do
  gem 'rvm-capistrano'
  gem 'capistrano'
end

group :production do
  gem 'unicorn'
end
{% endhighlight %}

Сapistrano будет отвечать за выполнение команд, а unicorn у нас будет в качестве сервера приложения.

Делаем

```
$ bundle install
$ git commit -am "add unicorn and capistrano gems"
```

И создаем файлы конфигурации для Capistrano:

{% highlight bash %}
$ capify .
[add] writing './Capfile'
[add] writing './config/deploy.rb'
[done] capified!
$ echo "/config/deploy.rb"  .gitignore
$ git commit -am "ignore deploy configuration"
{% endhighlight %}

Теперь нужно создать еще несколько конфигурационных файлов:

```
$ touch config/nginx.conf
$ touch config/unicorn.rb
$ touch config/unicorn_init.sh
```

* * *

#### Редактирование конфигурационных файлов 

/Capfile

{% highlight ruby %}    
load 'deploy'
load 'deploy/assets'
load 'config/deploy'
{% endhighlight %}
    

/config/deploy.rb

{% highlight ruby %}
require "rvm/capistrano"
require "bundler/capistrano"

set :application, "testapp"
set :shared_children, shared_children
set :repository,  "git@github.com:istickz/testapp.git"
set :deploy_to, "/var/www/testapp"
set :scm, :git
set :branch, "master"
set :user, "username"
set :group, "username"
set :use_sudo, false
set :rails_env, "production"
set :deploy_via, :copy
set :ssh_options, { :forward_agent => true, :port => 6629 }
set :keep_releases, 5
default_run_options[:pty] = true
server "XX.XX.XX.XX", :app, :web, :db, :primary => true

after "deploy", "deploy:cleanup"

namespace :deploy do
  %w[start stop restart].each do |command|
    desc "#{command} unicorn server"
    task command, roles: :app, except: {no_release: true} do
      run "/etc/init.d/unicorn_#{application} #{command}"
    end
  end

  task :setup_config, roles: :app do
    sudo "ln -nfs #{current_path}/config/nginx.conf /etc/nginx/sites-enabled/#{application}"
    sudo "ln -nfs #{current_path}/config/unicorn_init.sh /etc/init.d/unicorn_#{application}"
    run "mkdir -p #{shared_path}/config"
    put File.read("config/production_database.yml"), "#{shared_path}/config/database.yml"
    puts "Теперь вы можете отредактировать файлы в  #{shared_path}."
  end
  after "deploy:setup", "deploy:setup_config"

  task :symlink_config, roles: :app do
    run "ln -nfs #{shared_path}/config/database.yml #{release_path}/config/database.yml"
  end
  after "deploy:finalize_update", "deploy:symlink_config"

  desc "Make sure local git is in sync with remote."
  task :check_revision, roles: :web do
    unless `git rev-parse HEAD` == `git rev-parse origin/master`
      puts "WARNING: HEAD is not the same as origin/master"
      puts "Run `git push` to sync changes."
      exit
    end
  end
  before "deploy", "deploy:check_revision"

end
    
{% endhighlight %}

config/nginx.conf

```    
    upstream unicorn {
      server unix:/tmp/unicorn.testapp.sock fail_timeout=0;
    }
    
    server {
      listen 80 default deferred;
      server_name testapp.dev.istickz.ru;
      root /var/www/testapp/current/public;
    
      location ^~ /assets/ {
        gzip_static on;
        expires max;
        add_header Cache-Control public;
      }
    
    
      try_files $uri/index.html $uri @unicorn;
      location @unicorn {
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header Host $http_host;
        proxy_redirect off;
        proxy_pass http://unicorn;
      }
    
      error_page 500 502 503 504 /500.html;
      client_max_body_size 4G;
      keepalive_timeout 10;
    }
```    
    

config/unicorn.rb

{% highlight ruby %}
    root = "/var/www/testapp/current"
    working_directory root
    pid "#{root}/tmp/pids/unicorn.pid"
    stderr_path "#{root}/log/unicorn.log"
    stdout_path "#{root}/log/unicorn.log"
    
    listen "/tmp/unicorn.testapp.sock"
    worker_processes 2
    timeout 30
    
    # Force the bundler gemfile environment variable to
    # reference the capistrano "current" symlink
    before_exec do |_|
      ENV["BUNDLE_GEMFILE"] = File.join(root, 'Gemfile')
    end
{% endhighlight %}
    

config/unicorn_init.sh

{% highlight bash %}
    #!/bin/sh
    ### BEGIN INIT INFO
    # Provides:          unicorn
    # Required-Start:    $remote_fs $syslog
    # Required-Stop:     $remote_fs $syslog
    # Default-Start:     2 3 4 5
    # Default-Stop:      0 1 6
    # Short-Description: Manage unicorn server
    # Description:       Start, stop, restart unicorn server for a specific application.
    ### END INIT INFO
    set -e
    
    # Feel free to change any of the following variables for your app:
    TIMEOUT=${TIMEOUT-60}
    APP_ROOT=/var/www/testapp/current
    PID=$APP_ROOT/tmp/pids/unicorn.pid
    CMD="cd $APP_ROOT; bundle exec unicorn -D -c $APP_ROOT/config/unicorn.rb -E production"
    AS_USER=username
    set -u
    
    OLD_PIN="$PID.oldbin"
    
    sig () {
      test -s "$PID" && kill -$1 `cat $PID`
    }
    
    oldsig () {
      test -s $OLD_PIN && kill -$1 `cat $OLD_PIN`
    }
    
    run () {
      if [ "$(id -un)" = "$AS_USER" ]; then
        eval $1
      else
        su -c "$1" - $AS_USER
      fi
    }
    
    case "$1" in
    start)
      sig 0 && echo >&2 "Already running" && exit 0
      run "$CMD"
      ;;
    stop)
      sig QUIT && exit 0
      echo >&2 "Not running"
      ;;
    force-stop)
      sig TERM && exit 0
      echo >&2 "Not running"
      ;;
    restart|reload)
      sig HUP && echo reloaded OK && exit 0
      echo >&2 "Couldn't reload, starting '$CMD' instead"
      run "$CMD"
      ;;
    upgrade)
      if sig USR2 && sleep 2 && sig 0 && oldsig QUIT
      then
        n=$TIMEOUT
        while test -s $OLD_PIN && test $n -ge 0
        do
          printf '.' && sleep 1 && n=$(( $n - 1 ))
        done
        echo
    
        if test $n -lt 0 && test -s $OLD_PIN
        then
          echo >&2 "$OLD_PIN still exists after $TIMEOUT seconds"
          exit 1
        fi
        exit 0
      fi
      echo >&2 "Couldn't upgrade, starting '$CMD' instead"
      run "$CMD"
      ;;
    reopen-logs)
      sig USR1
      ;;
    *)
      echo >&2 "Usage: $0 <start|stop|restart|upgrade|force-stop|reopen-logs>"
      exit 1
      ;;
    esac
{% endhighlight %}

Не забываем сделать файл исполняемым.

```
$ chmod +x config/unicorn_init.sh 
```

Запушим конфигурационные файлы в репозиторий:

```
$ git add .
$ git commit "Add configuration files"
$ git push origin master
```

* * *

Конфигурационные файлы готовы, пора приступать к заливке приложения, но прежде давайте заново авторизуемся на сервере и выставим права на папку /var/www/ для пользователя username:

```
$ sudo chown -R username:username /var/www
$ sudo chmod -R g+w /var/www
```

Теперь можно спокойно сказать Capistrano, чтобы подготовило необходимую структуру папок на сервере и закинуло конфигурационные файлы куда нужно.

```
$ cap deploy:setup
```

Редактировать мы будем только доступы к базе данных. Сначала создадим базу данных и пользователя для нее.

```
$ mysql -u root -p
mysql> CREATE DATABASE `testapp` CHARACTER SET utf8 COLLATE utf8_general_ci;
mysql> GRANT ALL PRIVILEGES ON testapp.* TO testapp@localhost IDENTIFIED BY 'password' WITH GRANT OPTION;
```

Теперь редактируем доступы к БД.

```
$ cd /var/www/testapp/shared/config
$ nano database.yml 
```

И вписываем пользователя и пароль.

Ну что ж, теперь зальем наше приложение на сервер и скомпилируем ассеты:

```
$ cap deploy:cold
```

Готово! Теперь вы можете наблюдать за вашим приложением из браузера по ссылке http://yoursuperrailsapp.com

Осталось запускать unicorn сервер при каждом перезапуске дроплета.

```
$ sudo update-rc.d -f unicorn_testapp defaults
```

Теперь если вы изменяете код приложения, вам останется только запушить изменения в git репозиторий и сделать:

```
$ cap deploy
```

после чего все изменения уже будут на вашем сервере. На этом все. Исходный код приложения можно взять тут: [Github][7], а рабочий пример тут: http://testapp.dev.istickz.ru/

 [1]: https://www.digitalocean.com/
 [2]: https://pbs.twimg.com/media/BdJoINTCcAA6ha0.jpg ""
 [3]: https://cloud.digitalocean.com/ssh_keys
 [4]: https://cloud.digitalocean.com/droplets/new
 [5]: http://i2.wp.com/istickz.ru/wp-content/uploads/2014/02/do-panel.png
 [6]: https://github.com/capistrano/capistrano
 [7]: https://github.com/istickz/testapp
