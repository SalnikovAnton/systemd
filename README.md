# Инициализация системы. Systemd. 
1) Написать сервис, который будет раз в 30 секунд мониторить лог на предмет наличия ключевого слова. Файл и слово должны задаваться в /etc/sysconfig
2) Из epel установить spawn-fcgi и переписать init-скрипт на unit-файл. Имя сервиса должно также называться.
3) Дополнить Āнит-файл apache httpd возможностью запустить несколько инстансов сервера с разными конфигами

### 2 Устанавливаем spawn-fcgi и необходимые для него пакеты:
```
[root@otuslinux ~]# yum install epel-release -y && yum install spawn-fcgi php php-cli mod_fcgid httpd -y
```
раскомментируем строки с переменными в /etc/sysconfig/spawn-fcgi получится
```
[root@otuslinux ~]# cat /etc/sysconfig/spawn-fcgi
# You must set some working options before the "spawn-fcgi" service will work.
# If SOCKET points to a file, then this file is cleaned up by the init script.
#
# See spawn-fcgi(1) for all possible options.
#
# Example :
SOCKET=/var/run/php-fcgi.sock
OPTIONS="-u apache -g apache -s $SOCKET -S -M 0600 -C 32 -F 1 -P /var/run/spawn-fcgi.pid -- /usr/bin/php-cgi"
```
А сам юнит файл
```
[root@otuslinux ~]# cat /etc/systemd/system/spawn-fcgi.service
[Unit]
Description=Spawn-fcgi startup service by Otus
After=network.target

[Service]
Type=simple
PIDFile=/var/run/spawn-fcgi.pid
EnvironmentFile=/etc/sysconfig/spawn-fcgi
ExecStart=/usr/bin/spawn-fcgi -n $OPTIONS
KillMode=process

[Install]
WantedBy=multi-user.target
```
запуск и результат можно посмотреть командами
```
[root@otuslinux ~]# systemctl start spawn-fcgi
[root@otuslinux ~]# systemctl status spawn-fcgi
● spawn-fcgi.service - Spawn-fcgi startup service by Otus
   Loaded: loaded (/etc/systemd/system/spawn-fcgi.service; disabled; vendor preset: disabled)
   Active: active (running) since Sat 2023-02-04 17:11:52 UTC; 19h ago
 Main PID: 24291 (php-cgi)
    Tasks: 33 (limit: 5975)
   Memory: 18.3M
   CGroup: /system.slice/spawn-fcgi.service
           ├─24291 /usr/bin/php-cgi
           ├─24292 /usr/bin/php-cgi
           ├─24293 /usr/bin/php-cgi
           ├─24294 /usr/bin/php-cgi
           ├─24295 /usr/bin/php-cgi
           ├─24296 /usr/bin/php-cgi
           ├─24297 /usr/bin/php-cgi
           ├─24298 /usr/bin/php-cgi
           ├─24299 /usr/bin/php-cgi
           ├─24300 /usr/bin/php-cgi
           ├─24301 /usr/bin/php-cgi
           ├─24302 /usr/bin/php-cgi
           ├─24303 /usr/bin/php-cgi
           ├─24304 /usr/bin/php-cgi
           ├─24305 /usr/bin/php-cgi
           ├─24306 /usr/bin/php-cgi
           ├─24307 /usr/bin/php-cgi
           ├─24308 /usr/bin/php-cgi
           ├─24309 /usr/bin/php-cgi
           ├─24310 /usr/bin/php-cgi
           ├─24311 /usr/bin/php-cgi
           ├─24312 /usr/bin/php-cgi
           ├─24313 /usr/bin/php-cgi
           ├─24314 /usr/bin/php-cgi
           ├─24315 /usr/bin/php-cgi
           ├─24316 /usr/bin/php-cgi
           ├─24317 /usr/bin/php-cgi
           ├─24318 /usr/bin/php-cgi
```

### 3 будем использовать шаблон в конфигурации файла окружения /usr/lib/systemd/system/httpd.service. Копируем его в Админскую директорию /etc/systemd/system/
```
[root@otuslinux ~]# cp /usr/lib/systemd/system/httpd.service /etc/systemd/system/
```
и добавляем в файл строчку со значениями
```
EnvironmentFile=/etc/sysconfig/httpd-%I
```
создаем два файла конфига first.conf и second.conf из стандартного httpd.conf . Первый оставляем без изминений, а во втором меняем параметр Listen на 8080 и добавляем параметр PidFile /var/run/httpd-second.pid . Запускаем командами
```
[root@otuslinux ~]# systemctl start httpd@first
[root@otuslinux ~]# systemctl start httpd@second
```
результат и вывод можно посмотреть командой
```
[root@otuslinux ~]# ss -tnulp | grep httpd
tcp   LISTEN 0      128                *:8080            *:*    users:(("httpd",pid=27685,fd=4),("httpd",pid=27684,fd=4),("httpd",pid=27683,fd=4),("httpd",pid=27680,fd=4))
tcp   LISTEN 0      128                *:80              *:*    users:(("httpd",pid=27463,fd=4),("httpd",pid=27462,fd=4),("httpd",pid=27461,fd=4),("httpd",pid=27458,fd=4))
```



