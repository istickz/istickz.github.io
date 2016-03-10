---
title: Генератор картинок для групп Вконтакте
permalink: /ru/ruby-vk-image-generator/
categories: [Ruby]
tags: [ruby, vk]
---

Сегодня я расскажу, как нписать приложение для генерации картинок и постинга их в социальную сеть вконтакте.
<!--more-->
#### Генерация картинок. Для генерации картинок я буду использовать заранее подготовленные шаблоны(заготовки).

Следующий код будет добавлять текущую дату и содержание гороскопа:


```
require 'RMagick'
# date и description мы будем накладывать на заготовку
date =  "Четверг, 8 августа"
description = "Сегодня вашему любимому человеку будет непросто понять ваши истинные желания и намерения. Он будет стараться вывести их логическим путем, но верного результата никто гарантировать ему не сможет. Разве что вы сами смилостивитесь и подскажете."

# Создадим полотно и добавим на него нашу заготовку aquarius.jpg
canvas = Magick::ImageList.new("aquarius.jpg")

# Добавим дату на нашу заготовку
horo_date = Magick::Draw.new
horo_date.font_family = 'helvetica'
horo_date.pointsize = 16
horo_date.interline_spacing = 7
horo_date.annotate(canvas, 0,0,180,57, date) {
  self.fill = '#929da7'
}

# Добавим содержание гороскопа
horo_description = Magick::Draw.new
horo_description.font_family = 'helvetica'
horo_description.pointsize = 22
horo_description.interline_spacing = 7
horo_description.annotate(canvas, 0,0,190,100, description.place(40)) {
  self.fill = '#929da7'
}

# Запишем все наши изменения в новый файл.
canvas.write("test.jpg")
```    

На выходе получим вот это:

Наверное вы заметили, что в коде представленном выше есть метод place. Он нужен для разбивки и переноса строк содержимого гороскопа.

```
class String
  def place(count)
    words = self.split(" ") # Все слова из текста
    compozition = "" # Композиция - строка не превышающая count_sym символов
    summary = []  # Коллекция наших композиций

    words.each do |word|
      if (compozition.size + word.size + 1) <= count then
        compozition += " " + word
      else
        summary << compozition
        compozition = word
      end
    end
    # Добавим последний кусочек текста в summary который не обработался в цикле each
    summary << compozition
    # Уберем лишний пробел в начале
    summary.first[0] = ""

    return summary.join("\n")
  end
end
```    

#### Отправка на стену группы Вконтакте Отправлять картинки на стену мы будем используя gem vkontakte_api {#gemvkontakte_api}

```
require 'vkontakte_api'
public_id = 'ID группы вконтакте'
VkontakteApi.configure do |config|

  config.app_id       = VK_APP_ID
  config.app_secret   = VK_APP_SECRET
  config.adapter = :net_http
  config.http_verb = :post
  config.faraday_options = {
    ssl: {
      :verify => false
    },
  }
end

# Создадим клиента
app = VkontakteApi::Client.new(VK_ACCESS_TOKEN)

# Получим url для загрузки на стену
us = app.photos.get_wall_upload_server(gid: public_id)

# POST-запрос на полученный адрес
up = VkontakteApi.upload(url: us.upload_url, file1: ["Путь до изображения", 'image/jpeg'])

# Добавим описание
up.caption = "http://horo.ru"

# Добавляем gid к нашему up, в котором уже есть параметры server, hash и photo
up.gid = public_id

# Сохранение фотографии
save = app.photos.save_wall_photo(up)

# Отправка на стену группы
app.wall.post(attachments:save.first.id, owner_id: "-#{public_id}", from_group: 1 )
```    

Ну, генерация картинок у нас есть, отправка картинок тоже есть. Осталось только получать данные, которыми будем наполнять наши картинки.

#### Парсинг гороскопов Гороскопы я буду брать с сайта ignio.com, тем более что они предоставляют получение гороскопов по XML. Парсить XML будем с помощью Nokogiri Итак приступим. {#igniocomxmlxmlnokogiri}

```    
require 'nokogiri'
require 'open-uri'

url = "http://img.ignio.com/r/export/utf/xml/daily/lov.xml"
feed = Nokogiri::XML open(url)

feed.xpath("//aquarius//today")

# => [#<Nokogiri::XML::Element:0x123abf8 name="today" children=[#<Nokogiri::XML::Text:0x123a928 "\nНе ждите от вашего любимого человека слишком многого. Он не волшебник, он только учится, поэтому угадать, а тем более исполнить все ваши желания одновременно (а особенно те, которые вы не высказали ему лично) он просто не в состоянии. Будьте милосердны – и будет вам счастье.\n">]>]
```    

Ну, собственно и все. Тексты мы получили, генератор картинок у нас есть, и выгрузка в группы вконтакте у нас тоже имеется. Осталось только все автоматизировать. Исходники готового приложения можно посмотреть [тут][1]. А [тут][2] можно посмотреть, что получилось.

 [1]: https://github.com/istickz/vk-horo-poster
 [2]: http://vk.com/today_business_horoscope