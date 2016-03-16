---
title: Скрипт выхода из групп вконтакте
permalink: /ru/leave-vk-groups-script/
categories: ['Ruby', 'Туториалы', 'Социальные сети']
tags: ['ruby', 'vkontakte']

---


Сегодня я с ужасом обнаружил, что состою в 266 сообществах вконтакте! Это нужно было срочно исправить и выйти из ненужных мне сообществ.
<!--more-->
Я нашел несколько вариантов решения данной проблемы:

  1. Выходить самостоятельно.  
    Минусы: нужно заходить в каждую группу и нажимать на &#171;Выйти из группы&#187;, а это долго и мучительно&#8230; Плюсы: прокачаем мышцы рук
  2. Написать программу на Ruby.  
    Минусы: это отнимает 15+5 минут времени. Плюсы: прокачаем свои скиллы в Ruby

Ну что ж, 15 минут это не так уж и много, так давайте приступим к написанию программы.

Для доступа к vk api я буду использовать гем [vkontakte_api][1]

Для начала нужно будет создать приложение. Переходим по ссылке <http://vk.com/editapp?act=create>

[<img title="" src="http://i1.wp.com/istickz.ru/wp-content/uploads/2014/02/create_app.png?w=768" alt="create_app" data-recalc-dims="1" />][2]

Указываем имя и тип приложения. Далее нас попросят ввести код, который отправят по смс.  
Вводим его.

После чего добавляем описание и иконки:

[<img title="" src="http://i0.wp.com/istickz.ru/wp-content/uploads/2014/02/vk_leaver_info.png?w=768" alt="vk<em>leaver</em>info" data-recalc-dims="1" />][3]

Все, приложение готово. Осталось только взять ID нашего приложения и секретный ключ приложения.  
ID нужен для получения токена, а секретный ключ для работы самого приложения. Для этого преходим в настройки, забираем ID и начинаем создавать токен. Нам нужны права только для группы и, чтобы токен не имел срока годности, а это значит что нужно ввести в адресную строку следующую ссылку:

```
https://oauth.vk.com/authorize? 
 client_id=APP_ID& 
 scope=groups, offline& 
 redirect_uri=https://oauth.vk.com/blank.html& 
 display=page& 
 v=5.11& 
 response_type=token
```

Разрешаем приложению доступ и из адресной строки открывшейся страницы берем `access_token`

Ну, что ж, начнем писать наше приложение.

```ruby
require 'vkontakte_api'

VK_APP_ID = 'ID приложения'
VK_APP_SECRET = 'Защищенный ключ приложения'
VK_ACCESS_TOKEN = 'Токен пользователя'

VkontakteApi.configure do |config|
  config.app_id       = VK_APP_ID
  config.app_secret   = VK_APP_SECRET

  # По умолчанию логи запросов и ошибок пишутся в STDOUT, а нам это не нужно.
  # Отключаем логирование запросов
  config.log_requests = false
end

client = VkontakteApi::Client.new(VK_ACCESS_TOKEN)
# Получаем список групп пользователя
client_groups = client.groups.get
# Теперь будем перебирать все группы
# и спрашивать разреение на выход из группы.
# Шаблон вопроса
leave_question = "Выйти из группы %s?"
client_groups.each do |group_id|
  group = client.groups.get_by_id(group_ids: group_id).first
  puts "-"*50
  # Задаем вопрос
  puts leave_question % group.name

  # Получаем ответ
  print "ответ: "
  leave_answer = gets.chomp

  # Если пришел положительный ответ - "y" ,то выходим из группы,
  # если же пришел любой другой символ, то ничего не делаем
  if leave_answer == "y"
    client.groups.leave(group_id: group_id)
    puts "Вы успешно покинули группу."

  end
  puts "-"*50
end

```

Готово!  
Выполняем `$ ruby app.rb` из консоли и смело отвечаем на вопросы.

* * *

Внимание! Если у вас `require 'vkontakte_api` вывалился с ошибкой:

```
Gem::LoadError: Unable to activate faraday_middleware-0.9.0, because faraday-0.9.0 conflicts with faraday (&lt; 0.9, = 0.7.4)
```

То нужно установить gem faraday помладше, ну например версии: 0.8.9

* * *

P.S. Вот так вот Ruby может помогать в повседневной жизни.

 [1]: https://github.com/7even/vkontakte_api
 [2]: http://i1.wp.com/istickz.ru/wp-content/uploads/2014/02/create_app.png
 [3]: http://i0.wp.com/istickz.ru/wp-content/uploads/2014/02/vk_leaver_info.png
