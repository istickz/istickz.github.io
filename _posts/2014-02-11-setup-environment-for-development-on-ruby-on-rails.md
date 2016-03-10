---
title: Настройка окружения для разработки на Ruby on Rails
layout: post
permalink: /ru/setup-environment-for-development-on-ruby-on-rails/
categories: ['Ruby', 'Туториалы']
tags: ['ruby', 'ruby-on-rails']

---
Сегодня я покажу как из свежеустановленного linux сделать хорошую рабочую машинку готовую помочь вам разрабатывать приложения быстро, легко и непринужденно.
<!--more-->
Статья будет состоять из 5 частей.

  1. Установка Git
  2. Установка ZSH
  3. Установка RVM
  4. Установка Ruby, Rails,  
    4.1 Ruby  
    4.2 Rails
  5. Редакторы кода  
    5.1 Sublime Text 3  
    5.2 RubyMine

Итак, поехали&#8230;

## 1. Установка Git {#1git}

```$ sudo apt-get install git-core
```

Проверяем версию:

```
$ git --version
git version 1.8.3.2
```

Все отлично, давайте настроим git.

```
$ git config --global user.name "John Doe"
$ git config --global user.email johndoe@example.com
```

* * *

## 2. Установка ZSH {#2zsh}

```
$ sudo apt-get install zsh
```

Приступим к украшательству.

```
$ curl -L http://install.ohmyz.sh | sh
```

Все, украшательства и настройки zsh готовы.

Настало время переключиться на него. Это мы и сделаем:

```
$ sudo chsh -s $(which zsh) $(whoami)
```

После чего нужно будет перелогиниться и запустить консоль.

Иииха! Теперь у нас самый клевый терминал в мире.  
Сменить тему можно отредактировав строчку `ZSH_THEME` в `~/.zshrc`

Темы ищем здесь: https://github.com/robbyrussell/oh-my-zsh/wiki/themes

Сам проект тут: https://github.com/robbyrussell/oh-my-zsh

* * *

## 3. Установка RVM {#3rvm}

Ну что, потопали дальше?

<pre class="code code-shell-cmd" title="triple click to select command">➜ gpg --keyserver hkp://keys.gnupg.net --recv-keys 409B6B1796C275462A1703113804BB82D39DC0E3```

```
➜ \curl -sSL https://get.rvm.io | bash -s stable
```

Если словили такую ошибку:

```
* WARNING: Above files contains `PATH=` with no `$PATH` inside, this can break RVM,
    for details check https://github.com/wayneeseguin/rvm/issues/1351#issuecomment-10939525
    to avoid this warning append #PATH.
```

То лечим ее исправив файл `.zshrc`. А именно строки, содержащие export PATH=&#187;&#8230;&#187; заменяем на export PATH=$PATH:&#187;&#8230;&#187; .



Все, rvm готов. Можем перезапустить консоль, либо выполнить:

```
➜ source ~/.zshrc
```

* * *

## 4. Установка Ruby, Rails, Bundler {#4rubyrailsbundler}

### 4.1 Ruby {#41ruby}

Раньше бы мы стали запускать `rvm requirements` для установки зависимостей и отсутсвующих пакетов, но сейчас этого делать не нужно, т.к. rvm сам поймет, какие пакеты нужно поставить.

Поэтому, смело устанваливаем Ruby 2.0.0

```
➜ rvm install 2.1.1
```

Установка будет долгой, поэтому можете посмотреть, что творится за окном или сходить и приготовить себе кружку бодрящего кофе.

Скажем системе, что ruby 2.0.0 у нас будет использоваться по умолчанию.

```
➜ rvm use 2.1.1 --default
```

И еще я советую сделать еще один хук. На нашей рабочей машине хранить документацию незачем, мы всегда можем найти ее в интернете. Поэтому в файл `.gemrc` закинем строку &#171;gem: &#8212;no-rdoc &#8212;no-ri&#187;.

```
➜ echo "gem: --no-rdoc --no-ri"  ~/.gemrc
```

### 4.2 Rails {#42rails}

Установка всего в одну команду.

```
➜ gem install rails
```

* * *

## 5. Редакторы кода {#5}

В основном я использую Sublime Text 3, но недавно перешел на RubyMine 6.  
Давайте установим оба редактора.

К сожалению, продукты платные и в скором времени, возможно, Вам придется задуматься о приобретении лицензии на пользование данными продуктами.

### 5.1 Sublime Text 3 {#51sublimetext3}

Установка очень простая. Переходим на сайт http://www.sublimetext.com/3 Скачиваем версию под вашу платформу. И запускаем файл. Дожидаемся окончания установки и вуаля, все готово.

### 5.2 RubyMine {#52rubymine}

C RubyMine придется немного повозиться, но в конце мы получим полноценный редактор кода.

Перейдите на сайт http://www.jetbrains.com/ruby/ и скачайте последнюю версию продукта.

Распакуйте скачанный архив в домашнюю директорию, например в ~/apps/RubyMine. Далее, просто запустите rubymine.sh файл из папки RubyMine/bin/ и следуйте инструкциям.

После завершения установки вы сможете запускать RubyMine из меню приложений. Но без Java ничего не запустится, поэтому давайте поставим Java. Провереям версию Java.

```
$ java -version
```

Если мы увидим Open JDK, то незамедлительно сносим его с нашей системы.

```
$ sudo apt-get remove openjdk*
```

Теперь установим Oracle Java 7.

```
sudo add-apt-repository ppa:webupd8team/java
sudo apt-get update
sudo apt-get install oracle-java7-installer
```

После установки проверяем версию JAVA и компилятора Java.

```
➜ java -version
java version "1.7.0_51"
➜ javac -version 
javac 1.7.0_51
```

Отлично, версии совпадают. Осталось только запустить RubyMine и начать разрабатывать приложения.

* * *

## The End {#theend}

Поздравляю, у нас в руках самая свежая версия редактора RubyMine и он доступен в списке наших приложений.

На этом все, можно сказать что мы хорошо настроили нашу машинку для разработки Ruby on Rails приложений. Мне данной конфигурации вполне хватает и, вам я думаю, тоже такая конфигурация подойдет. Но как говорится, не предела совершенству. Пробуйте, дерзайте, творите!

Источники:

<http://git-scm.com/book/ru/Введение-Установка-Git>

<http://blog.coolaj86.com/articles/zsh-is-to-bash-as-vim-is-to-vi.html>

<https://github.com/zsh-users/zsh/blob/master/INSTALL>

<http://www.linuxrussia.com/2013/04/oracle-java-7-ubuntu-1304-1204-1210.html>