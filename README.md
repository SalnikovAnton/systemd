# Инициализация системы. Systemd. 
1) Написать сервис, который будет раз в 30 секунд мониторить лог на предмет наличия ключевого слова. Файл и слово должны задаваться в /etc/sysconfig
2) Из epel установить spawn-fcgi и переписать init-скрипт на unit-файл. Имя сервиса должно также называться.
3) Дополнить Āнит-файл apache httpd возможностью запустить несколько инстансов сервера с разными конфигами










#### 3 будем использовать шаблон в конфигурации файла окружения /usr/lib/systemd/system/httpd.service. Копируем его в Админскую директорию /etc/systemd/system/
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



