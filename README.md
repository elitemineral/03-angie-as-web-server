# Angie как веб-сервер

1. default.conf

```
server {
 listen 8080 default_server;
 server_name _;
 return 444;
}
```

2. site.conf

```
map $http_accept $preferred_ext {
    "~avif"   ".avif";
    "~webp"   ".webp";
    default "";
}

server {
    listen 8080;
    server_name site-old.ru www.site-old.ru;

    return 301 http://site.ru:8080$request_uri;
}

server {
    listen 8080;
    server_name site.ru;
    root /var/www/html/site;
    index index.html;

    location / {
        try_files $uri $uri/ =404;
    }

    # Изолированный блок для статики
    location ^~ /assets/ {
        expires 30d;
        access_log off;
        add_header Cache-Control "public";
    }

    # Регулярные выражения для медиа-файлов
    location ~* ^(?<base>.+)\.(ico|gif|png|jpe?g)$ {
        try_files $base$preferred_ext $uri =404;
        expires 30d;
        access_log off;
    }

    # Обработка ошибок
    error_page 404 /error/index.html;
    location = /error/index.html {
        internal;
    }
}
```

![alt text](site.png)