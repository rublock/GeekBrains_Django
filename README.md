# Django_GeekBrains

## Стек

- Python 3.8.10
    - isort, black, autoflake
    - Django < 3.3
    - Celery[Redis]
-VSCode
-SQLite 3

## Лицензия
MIT

## Настройка деплоя

установка переменного окружения ОС
```
export DJANGO_SETTINGS_MODULE=config.settings
```
проверка переменного окружения 
```
env | grep DJANGO
```
заходим на удаленный сервер
```
ssh root@194.67.87.131
```
создаем пользователя
```
adduser [username]
```
добавляем права sudo
```
adduser [username] sudo
```
выходим из сервера
```
logout
```
заходим под новым пользователем
```
ssh [username]@194.67.87.131
```
генерируем ssh ключи
```
ssh-keygen -b 4096 -C ["email"]
```
перенести ssh ключ на удаленный сервер (комманда выполняется с локального компьютера!)
чтобы заходить на сервер без запроса пароля
```
ssh-copy-id [username]@194.67.87.131
```
настройка вполнения sudo
```
hostname
```
```
sudo nano /etc/hosts
```
```
127.0.0.1 194.67.87.131
```
обновить все компоненты
```
sudo apt upgrade && sudo apt update -y
```
установка БД PostgreSQL
```
sudo apt-get install -y postgresql-contrib
```
запомнить какая верстия PostgreSQL установлена
```
pg_config --version
```
заходим в консольный клиент PostgreSQL для создания БД
```
sudo -u postgres psql
```
создаем БД
```
CREATE DATABASE [db name];
```
создаем пользователя
```
CREATE USER [user name] with NOSUPERUSER PASSWORD ['password'];
```
устанавливаем привелегии
```
GRANT ALL PRIVILEGES ON DATABASE [db name] TO [user name];
```
устанавливаем кодировку
```
ALTER ROLE [user name] SET CLIENT_ENCODING TO 'UTF8';
```
изоляция для транзакций
```
ALTER ROLE [user name] SET default_transaction_isolation TO 'READ COMMITTED';
```
выход из psql
```
ctrl + Z
```
запуск кластера БД
```
sudo pg_ctlcluster [db version] main start
```
перенос файлов, комманда выполняется локально!
```
rsync --archive --compress --delete . admin@194.67.87.131:/home/admin/GeekBrains_Django_project
```
устанавливаем менеджер пакетов pip
```
apt install python3-pip
```
устанавливаем на сервер витуальное окружение
```
sudo pip install virtualenv
```
создаем виртуальное окружение
```
virtualenv venv
```
установка зависимосте
```
pip install -r requirements.txt
```
установка переменного окружения на сервере и проверка
```
export DJANGO_SETTINGS_MODULE=config.conf_prod && env | grep DJANGO
```
выполняем миграции
```
python manage.py migrate
```
загружаем фикстуры
```
python manage.py loaddata 001_news 002_courses 003_lessons 004_teachers 001_user_admin
```
Сборка всех дополнительных статичных файлов
```
python manage.py collectstatic
```
запускаем gunicorn
```
gunicorn config.wsgi
```
в новом терминале на сервере проверяем работу gunicorn через curl должно быть 200 OK
```
curl -i http://127.0.0.1:8000/mainapp/ | head -n 25
```
Создание конфигурации сервиса gunicorn
```
sudo nano /etc/systemd/system/gunicorn.service
```
Конфигурация сервиса
```
[Unit]
Description=Gunicorn daemon for [project name]
After=network.target

[Service]
User=[user name]
Group=[group name]
WorkingDirectory=[path to project]
ExecStart=[path to project]/venv/bin/gunicorn --access-logfile - --workers 3 --env DJANGO_SETTINGS_MODULE="config.conf_prod" --env DJANGO_SECRET_KEY='_h94+q_xj7dshrj58iyznl^ighi=qlffj!d2jncd87nzkucke1' --bind unix:[path to project]/[file name].sock config.wsgi

[Install]
WantedBy=multi-user.target
```
Добавление сервиса в автозагрузку
```
sudo systemctl enable gunicorn
```
Запуск сервиса
```
sudo systemctl start gunicorn
```
Проверка статуса сервиса
```
sudo systemctl status gunicorn
```
устанавливаем nginx
```
sudo apt install -y nginx
```
Создание конфигурации сервиса nginx
```
sudo nano /etc/nginx/sites-available/[name of configuration]
```
добавляем в файл конфигурацию
```
server {
    listen 80;
    server_name 194.67.87.131; //либо домен
    location = /favicon.ico
    
    {
        access_log off;
        log_not_found off;
    }

    location /static/ {
        root [path to project];
    }

    location /media/ {
        root [path to project];
    }

    location / {
        include proxy_params;
        proxy_pass http://unix:[path to project]/[file name].sock;
    }
}
```
Включение новой конфигурации сервиса Nginx
```
sudo ln -s /etc/nginx/sites-available/[name of configuration] /etc/nginx/sites-enabled/[name of configuration]
```
Проверка Nginx
```
sudo nginx -t
```
Перезапуск сервера nginx чтобы перечиталась конфигурация
```
sudo systemctl restart nginx
```
Проверка статуса сервиса nginx
```
sudo systemctl status nginx
```
Установка Redis
```
sudo apt install redis-server
```
Конфигурация сервиса Redis
```
sudo nano /etc/redis/redis.conf
```
Пишем конфигурацию чтобы сервис редиса подчитанялся планировщику системди
```
supervised systemd
```
Перезапуск сервиса redis
```
sudo systemctl restart redis-server
```
Проверка сервиса redis
```
sudo systemctl status redis-server
```
Создание конфигурации сервиса celery
```
sudo nano /etc/systemd/system/celeryd.service
```
файл конфиграции селери
```
[Unit]
Description=Celery daemon for [project name]
After=network.target

[Service]
User=[user name]
Group=[group name]
WorkingDirectory=[project path]
Environment='DJANGO_SETTINGS_MODULE=config.conf_prod'
'DJANGO_SECRET_KEY=_h94+q_xj7dshrj58iyznl^ighi=qlffj!d2jncd87nzkucke1'
ExecStart=[project path]/venv/bin/celery -A config worker -l INFO

[Install]
WantedBy=multi-user.target
```
Добавление сервиса селери в автозагрузку
```
sudo systemctl enable celeryd
```
Запуск сервиса селери
```
sudo systemctl start celeryd
```
Проверка статуса сервиса
```
sudo systemctl status celeryd
```
Установка пакета фаервола
```
sudo apt install -y ufw
```
Проверка доступных приложений фаервола
```
sudo ufw app list
```
Включение ufw фаервола
```
sudo ufw enable
```
Включение правил для ufw фаервола или https
```
sudo ufw allow 'Nginx HTTP';
```
```
sudo ufw allow 'OpenSSH';
```
Проверка статуса ufw фаервола
```
sudo ufw status
```
может выскочить ошибка 502 Bad Gateway
отключить gunicorn
```
systemctl stop gunicorn
```
удалить службу из автозагрузки gunicorn
```
systemctl disable gunicorn
```
подбробнее об ошибке можно почитать тут

gunicorn слушает порт 8000
gunicorn --bind 0.0.0.0:8000 config.wsgi
но ваша конфигурация NGINX отправляет запросы в сокет
proxy_pass http://unix:/run/gunicorn.sock;
так что либо сделайте это, чтобы заставить gunicorn слушать этот сокет
gunicorn --bind unix:/run/gunicorn.sock config.wsgi
или сделайте это, чтобы заставить NGINX отправлять запросы на порт 8000 на локальном хосте
proxy_pass http://localhost:8000

если нет соединения с базой данных проверить есть ли сама база и юзер с правильнм имененм и паролем в PosgresSQL