#### Написание Dockerfile для nginx
Файл можно найти в репозитории 
[Dockerfile-nginx](https://github.com/xeniaweber/otus/blob/test/hw18/Dockerfile-nginx)

#### Build образа из Docker файла
Из данного файла был собран образ
```console
docker build -t nginx_otus:hw18 -f Dockerfile-nginx .
```
#### Push образа в репозиторий docker hub
Созданный образ был запушен в репозиторий 
```console
docker tag nginx_otus:hw18 xweber/otus:hw18
docker push xweber/otus:hw18
```
Ссылка на образ:
[xweber/otus:hw18](https://hub.docker.com/layers/xweber/otus/hw18/images/sha256-0b1f218e412e482881a0126dee21e5b3b09747e3131e16c290f94a343be55702?context=explore)

#### Итог
Запустив данный образ и введя в строку браузера http://localhost:80 можно увидеть сообщение "Hello, Otus!"
