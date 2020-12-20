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
docker tag otus:nginx_hw18 xweber/otus:nginx_hw18
docker push xweber/otus:nginx_hw18
```
Ссылка на образ:
[xweber/otus:nginx_hw18](https://hub.docker.com/layers/xweber/otus/nginx_hw18/images/sha256-fe02648e189eb11ed5fd2c48ff6b64de061699a9ddf049b146ced22bb19a12f6?context=explore)

#### Итог
Запустив данный образ и введя в строку браузера http://localhost:80 , можно увидеть сообщение "Hello, Otus!"
