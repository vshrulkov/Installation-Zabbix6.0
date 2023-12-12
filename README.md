# Installation-Zabbix6.0
Installation Zabbix6.0 on Ubuntu20
_В данной инструкции установка будет осуществлятся на веб-сервере "nginx "_
_Отличие от "Apache" только в том, что в "nginx" к удаленному веб-сервереу будем обращатся - __http://<ip-сервера:порт>__. В "Apache" обращение к удаленному веб-сервереу будет - __http://<ip-сервера:порт>/папка с проектом__._
_В Ubuntu 20.04 компонент "Apache" уже установлен, поэтому в дальнейшем его удалим во избежании конфликтов веб-серверов._
___
1. Вход с правами суперпользователя -> вводим пароль
```
sudo su
```
2. Адрес репозитория с Zabbix
```
wget https://repo.zabbix.com/zabbix/6.0/ubuntu/pool/main/z/zabbix-release/zabbix-release_6.0-1+ubuntu20.04_all.deb
```
3. Распаковка репозитория
```
dpkg -i zabbix-release_6.0-1+ubuntu20.04_all.deb
```
4. Обновим пакеты после распаковки
```
apt update
```
5. Устанавливаем Zabbix сервер. Так же установится фронтент Zabbix, sql скрипты для Zabbix.
```
apt install zabbix-server-pgsql zabbix-frontend-php php7.4-pgsql zabbix-nginx-conf zabbix-sql-scripts zabbix-agent
```
6. Устанавливаем серев PostgreSQL
Адрес репозитория PostgreSQL 
```
wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -
```
Ключи для скачивания 
```
echo "deb http://apt.postgresql.org/pub/repos/apt/ `lsb_release -cs`-pgdg main" |sudo tee  /etc/apt/sources.list.d/pgdg.list
```
Обновляем пакеты
```
sudo apt update
```
Установка PostgreSQL13.  __Пимечание__: _Данная верися Zabbix работает с версией PostgreSQL13. Если в команде указать без версии, тогда устанвится самая последняя версия и работать с версией Zabbix 6 не будет._
```
sudo apt install postgresql-13
```
Проверяем, что сервер PostgreSql запущен (active)
```
systemctl status postgresql
```
Создаем пользователя на сервере PostgreSql с именем "zabbix". _В ходе создания будет запрошен пароль, необходимо его задать и запомнить._
```
sudo -u postgres createuser --pwprompt zabbix
```
Создаем базу данных
```
sudo -u postgres createdb -O zabbix zabbix
```
Выполняем экспорт в созданную базу данных sql скриптов из ранее скачанного архива
```
zcat /usr/share/zabbix-sql-scripts/postgresql/server.sql.gz | sudo -u zabbix psql zabbix
```
Заходим текстовым редактором в файл конфигурации zabbix и находим там значение 'DBPassword=' - убираем # - знак комментария и вводим наш пароль от базы данных. Сохраняем.

_Первый вариант редактирования (консоле)_
```
nano /etc/zabbix/zabbix_server.conf 
```
_Второй вариант редактирования (в текстовом редакторе)_
```
gedit /etc/zabbix/zabbix_server.conf 
```
7. Настраиваем файл конфигурации nginx
```
nano /etc/zabbix/nginx.conf 
```
_В самом начале файла убераем знак комментария # и вводим наш адрес локальной сети._
```
listen 80;
server_name 127.0.0.1; 
```
8. Удаляем сервер "Apache"
```
sudo apt remove apache2
```
9. Перезагружаем службу заббикс сервера
```
systemctl restart zabbix-server zabbix-agent nginx php7.4-fpm
```
10. Добавляем в автозапуск
```
systemctl enable zabbix-server zabbix-agent nginx php7.4-fpm
```
11. По адресу http://<ip адрес сервера:порт> - откроется стартовая страница Zabbix 6
