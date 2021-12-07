# Приложение  
Образ с приложением загружен на hub.docker.com.   
Доступен по адресу [nginx:alpineapp](https://hub.docker.com/r/alsxs/nginx).  
  
Содержимое Dockerfile:
```shell
FROM nginx:1.21.4-alpine
WORKDIR  /usr/share/nginx/html
ADD index.html /usr/share/nginx/html/
ADD style.css /usr/share/nginx/html/
```