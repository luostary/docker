# docker
Инструкции
**Установка XDEBUG версии 2**

Эта инструкция настраивает слой "XDEBUG" для текущего контейнера с установленной версией php - 7.1

0. Переходим в папку dockerTender 

---

1. Создаем файл Dockerfile  с таким содержимым

```
FROM YOUR_CURRENT_IMAGE
#Install xdebug 2.9.8
RUN curl -O -k https://xdebug.org/files/xdebug-2.9.8.tgz && tar -xzf xdebug-2.9.8.tgz
RUN cd /xdebug-2.9.8 && phpize
RUN cd /xdebug-2.9.8 && ./configure --enable-xdebug && make && make install
```
---

2. С помощью файла создаем новый образ

`sudo docker build . --tag new-image

---

3. Создаем файл xdebug.ini и кладём его в папку с конфигами server/config/xdebug.ini
Такое содержимое 
```
[xdebug]
;version 2
xdebug.remote_enable=1
;xdebug.idekey=PHPSTORM
;xdebug.remote_log=/xdebug.log
xdebug.remote_mode=req
xdebug.remote_connect_back=1

; Можно настроить переход из ошибки в trace в файл в IDE
xdebug.file_link_format = phpstorm://open?%f:%l

;version 3
zend_extension=xdebug.so
xdebug.client_host=localhost
xdebug.client_port=9002
xdebug.mode=debug
xdebug.discover_client_host=1
```
---

4. В файле docker-compose.yml в блоке web: image: Меняем образ на новый, который указали в пункте (2) new-image
```
image: new-image
```
В блоке volumes: мапим файл настроек xdebug.ini
```
- ./server/config/xdebug.ini:/etc/php/7.1/apache2/conf.d/xdebug.ini
```

---

5. далее перезапускаем веб-сервер. (Чтобы не потерять "нажитую" схему баз данных, перезапускаем только конкретно веб-контейнер)

`sudo docker-compose stop web && sudo docker-compose rm -f web && sudo docker-compose up -d web`

--

Теперь в таком же как и раньше окружении у нас должен появиться установленнй xdebug
Чтобы проверить это, вызываем версию веб-сервера

`sudo docker exec apacheServer sh -c 'php -v'`

Это только подготовка, после этого еще требуется настроить XDEBUG в вашей IDE
