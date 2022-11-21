Образы докер с xdebug, oci8 и другими расширениям
Заупуск

```console
docker run --name php-73 -p 127.0.0.1:9173:9000 -v /var/www:/var/www --add-host=host.docker.internal:host-gateway -d php-with-extensions:7.3-fpm
```

или

```console
docker-compose up -d
```

Заупук консольных скриптов

```console
docker exec -it php-72 php /var/www/html/protected/yiic test
```

Если не работает xdebug и в фаерволе по стоит INPUT DROP - добавить строчку

```console
iptables -A INPUT ! -i eno1  -d 172.17.0.0/16 -j ACCEPT
```
