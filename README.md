# Деплой Node.js, с уcтановкой Nginx и SSL Let's Encrypt

[![DigitalOcean Referral Badge](https://web-platforms.sfo2.cdn.digitaloceanspaces.com/WWW/Badge%201.svg)](https://www.digitalocean.com/?refcode=96eb2d860a30&utm_campaign=Referral_Invite&utm_medium=Referral_Program&utm_source=badge)


 ## Установка необходимых библиотек

- Подключитесь к серверу по SSH.
- Обновите состояние пакетов:

```
sudo apt update
```
- Установите NVM (Node Version Manager):

```
curl https://raw.githubusercontent.com/creationix/nvm/master/install.sh | bash
source ~/.profile 
```
- Установите Node:

```
# Установить последную версию
nvm install node
# Установить конкретную версию
nvm install 14.17.3
nvm use 14.17.3
# Проверить активную версию 
node -v
```
- Склонируйте Git репозиторий:

```
git clone (https ссылка на репозиторий)
```
- Зайдите в папку приложения и установите node_modules:

```
ls -l # Посмотреть список файлов
cd (имя репозитория)
npm install
```
- Установите PM2 и запустите node приложение:

```
npm install -g pm2
NODE_ENV=production pm2 start npm --name strapi -- run start # Запустить в режиме продакшн npm run start скрипт и назвать "strapi"
pm2 status # Статус процессов
pm2 logs # Показать логи приложения (Ctrl + C чтобы выйти)
pm2 startup ubuntu # Запускать pm2 при рестарте системы
pm2 save # Сохранить процесс чтобы при перезапуске сам запускался
```
## Установка firewall (в google cloud firewall можно настроить из веб интерфейса)

```
ufw status
ufw enable # Oтвечаем 'y'
ufw allow ssh
ufw allow http
ufw allow https
```
## Установка и базовая настройка Nginx 
```
sudo apt install nginx # Отвечаем 'y'
cd /etc/nginx/sites-enabled
sudo nano your-domain.com
```
- Пишем максимально базовый конфиг

```
server
{
	listen 80;
	root /var/www/your-domain.com
}
```
- Далее создаем в  /var/www/your-domain.com файл index.html

```
cd  /var/www
sudo mkdir /your-domain.com
cd your-domain.com/
sudo nano index.html
```
- со следующим содержимым

```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>

</head>
<body>
<div>Hello, andrey</div>
</body>
</html>
```
- Таким образом мы настроили очень базово настроили nginx, можем проверить перейдя по адресу нашего домена

## Получение lets encrypt сертификата
```
sudo apt install certbot python3-certbot-nginx # Отвечаем 'y'
sudo certbot -a webroot -i nginx # отвечаем на вопрос
```
- Таким образом мы должны получить сертификат. Далее, если мы хотим настроить reverse proxe для nodejs например, то надо добавить блок location

```
cd /etc/nginx/sites-enabled
sudo nano your-domain.com
```
- Ты увидишь

```
server
{
        root /var/www/frolov.store;
        server_name frolov.store;


    listen 443 ssl; # managed by Certbot
    ssl_certificate /etc/letsencrypt/live/frolov.store/fullchain.pem; # managed by Certbot
    ssl_certificate_key /etc/letsencrypt/live/frolov.store/privkey.pem; # managed by Certbot
    include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot
}
server
{
    if ($host = frolov.store) {
        return 301 https://$host$request_uri;
    } # managed by Certbot


        listen 80;
        server_name frolov.store;
    return 404; # managed by Certbot


}

```
- Добавляем location/ {**}

```
server
{
        root /var/www/frolov.store;
        server_name frolov.store;


    listen 443 ssl; # managed by Certbot
    ssl_certificate /etc/letsencrypt/live/frolov.store/fullchain.pem; # managed by Certbot
    ssl_certificate_key /etc/letsencrypt/live/frolov.store/privkey.pem; # managed by Certbot
    include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot
 location / {
                proxy_pass http://localhost:3000;
                                proxy_http_version 1.1;
                                proxy_set_header Upgrade $http_upgrade;
                                proxy_set_header Connection 'upgrade';
                                proxy_set_header Host $host;
                                proxy_set_header X-Real-IP $remote_addr;
                                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                                proxy_set_header X-NginX-Proxy true;
                                proxy_cache_bypass $http_upgrade;
        }
}
server
{
    if ($host = frolov.store) {
        return 301 https://$host$request_uri;
    } # managed by Certbot


        listen 80;
        server_name frolov.store;
    return 404; # managed by Certbot


}

```
- Ctrl + X чтобы выйти, 'y' чтобы сохранить
- Если запущен Apache сервер, его необходимо остановить
```
service apache2 status
sudo systemctl disable apache2 && sudo systemctl stop apache2
```
- Проверьте синтаксис Nginx и перезапустите сервер
```
sudo nginx -t
sudo service nginx restart
```

## Привязка домена
- Добавьте домен в DigitalOcean (страница Networking, вкладка Domains).
- Добавьте две записи типа 'A'. В первой Hostname равен '@', во второй 'www'. Нажмите на поле с IP и выберете droplet (сервер).
- На странице регистратора вашего домена добавьте DNS записи:
```
ns1.digitalocean.com
ns2.digitalocean.com
ns3.digitalocean.com
```
- Подождите до 48 часов (может быть и 5 минут) пока обновятся DNS записи

## Проверка автоматического обновления сертификата
```
sudo certbot renew --nginx --dry-run
```

**Да будет свет**
