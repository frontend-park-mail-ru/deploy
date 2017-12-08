# Деплоим фронтенд 🏆

Дано:
1. Домен
2. Купленный сервак/винтуалка (amazon, vscale, digitalocean)

Первым делом настроим сборку фронтенда через travis-ci и его доставку на сервер.
Для сборки соорудим простенький  .travis-ci.yml файлик:

```
language: node_js
node_js:
- '8'
script:
- npm run build
```

А теперь немного усложним и настроим копирование собранных файлов на сервер. Для этого надо как-то поместить в тревис наш приватный ключ.
Это можно сделать командой `travis encrypt-file ~/.ssh/technopark --add`

Тревис зашифрует файл и положит часть файла и ключ для расшифровки у себя, а в репозиторий добавит файлик technopark.enc. Его надо не забыть запушить.

Итого, чтобы положить файлы на сервер:

```
language: node_js
node_js:
- '8'
script:
- eval "$(ssh-agent -s)"
- ssh-keyscan -H $SERVER_IP >> ~/.ssh/known_hosts
- chmod 600 $HOME/.ssh/technopark
- ssh-add $HOME/.ssh/technopark
- npm run build
- scp -r ./dist root@$SERVER_IP:/var/www
before_install:
- openssl aes-256-cbc -K $encrypted_cff8668a0280_key -iv $encrypted_cff8668a0280_iv
  -in technopark.enc -out $HOME/.ssh/technopark -d
```

После всех манипуляций можно устанавливать nginx:

`yum install nginx` (Centos, Fedora, RHEL)

`apt-get install nginx` (Debian, Ubuntu, чтотоещё)

⚠️ Важно! Отключите SeLinux командой `setenforce 0`, иначе можете получать вечные 403 ошибки (актуально для centos)

Настроим nginx конфиг (/etc/nginx/conf.d/default.conf)
```
server {
  listen 80;
  server_name jsdaddy.tech;

  location / {
    root /var/www;
  }
}
```

Ну и всё работает. Не хватает https. Его можно сделать бесплатно с помощью letsencrypt.
Как это работает:
1. Мы ставим на сервак специальное приложение certbot (или запускаем в докере)
2. Запускаем certbot в одном из режимов (webroot или standalone)

    2.1. В режиме webroot certbot генерирует файлик и кладёт его в директорию, которую мы указываем как webroot. Этот файл должен быть доступен по пути `http://<ваш_домен>/.well-known/acme-challenge/<какой-то файл>`. Certbot после генерации файла дёргает ваш его по публичному пути, таким образом проверяет, что вы - реальный владелец сайта и выписывает сертификат. Сертификаты кладутся в директорию `/etc/letsencrypt/live/вашдомен/fullchain.pem`

    Nginx конфиг до получения сертификата:

    ```
    server {
      listen 80;
      server_name jsdaddy.tech;

      location / {
        root /var/www;
      }

      location /.well-known/acme-challenge/ {
        root /var/www/webroot;
      }
    }
    ```

    Получаем сертификат: `certbot certonly --webroot -w /var/www/webroot -d jsdaddy.tech`

    2.2. В режиме standalone нам нужно остановить nginx, потому что certbot поднимет свой сервер для проверки владельца на 80 порту: `certbot certonly --standalone -d jsdaddy.tech`

    Используйте этот способ только если не боитесь даунтайма вашего сервиса.

    ⚠️ У letsencrypt есть лимит - 20 сертификатов на домен за 3 часа. Не заиграйтесь :)

После получения сертификата можно уже врубать его для сайта и редиректить с http версии.

```
    server {
      listen 80;
      server_name jsdaddy.tech;

      location / {
        return 301 https://jsdaddy.tech$request_uri;
      }

      location /.well-known/acme-challenge/ {
        root /var/www/webroot; # оставим это для обновления сертификата
      }
    }


    server {
      listen 443 ssl http2;
      server_name jsdaddy.tech;

      ssl_certificate /etc/letsencrypt/live/jsdaddy.tech/fullchain.pem;
      ssl_certificate_key /etc/letsencrypt/live/jsdaddy.tech/privkey.pem;

      location / {
        root /var/www;
      }
    }
```


