```
Задание
- Запретить всем пользователям, кроме группы admin логин в выходные (суббота и воскресенье), без учета праздников
```
1. Создаю пользователей, группу admin; добавлю некоторых локальных пользователей в группу
```
user@user-VirtualBox:~/auth$ vagrant ssh
Last login: Thu Feb  1 17:59:59 2024 from 10.0.2.2
[vagrant@pam ~]$ sudo -i
[root@pam ~]# useradd otusadm && useradd otus
[root@pam ~]# echo "OTUS2024!" | passwd --stdin otusadm && echo "OTUS2024!" | passwd --stdin otus
Changing password for user otusadm.
passwd: all authentication tokens updated successfully.
Changing password for user otus.
passwd: all authentication tokens updated successfully.
[root@pam ~]# groupadd -f admin
[root@pam ~]# usermod otusadm -a -G admin && usermod vagrant -a -G admin && usermod root -a -G admin
[root@pam ~]# cat /etc/group | grep admin
admin:x:1003:otusadm,vagrant,root
```
Подключаюсь по ССШ к виртуальной машине с хостовой машины с помощью созданных пользователей для проверки
```
user@user-VirtualBox:~/auth$ ssh otus@192.168.57.10
otus@192.168.57.10's password: 
Last login: Thu Feb  1 18:21:12 2024 from 192.168.57.1
[otus@pam ~]$ whoami
otus
[otus@pam ~]$ exit
logout
Connection to 192.168.57.10 closed.
user@user-VirtualBox:~/auth$ ssh otusadm@192.168.57.10
otusadm@192.168.57.10's password: 
Last login: Thu Feb  1 18:22:23 2024 from 192.168.57.10
[otusadm@pam ~]$ whoami
otusadm
[otusadm@pam ~]$ exit
logout
Connection to 192.168.57.10 closed.
```
Создаю скрипт контроля доступа в выходные дни, назначаю права на исполнение
```
[root@pam ~]# touch /usr/local/bin/login.sh
[root@pam ~]# nano /usr/local/bin/login.sh
#!/bin/bash
if [ $(date +%a) = "Sat" ] || [ $(date +%a) = "Sun" ]; then
 if getent group admin | grep -qw "$PAM_USER"; then
        echo "group admin"
        exit 0
      else
        echo "user's can't connect in weekend"
        exit 1
    fi
  else
    echo "welcome all"
    exit 0
fi

[root@pam ~]# chmod +x /usr/local/bin/login.sh
```
Далее прописываю в /etc/pam.d/sshd запуск скрипта с помощью модуля pam_exec
```
[root@pam ~]# nano  /etc/pam.d/sshd
...
session    required     pam_exec.so /usr/local/bin/login.sh
...
```
Дожидаюсь выходного дня и проверяю результат
```
user@user-VirtualBox:~/auth$ ssh otus@192.168.57.10
otus@192.168.57.10's password: 
/usr/local/bin/login.sh failed: exit code 1
Connection closed by 192.168.57.10 port 22

user@user-VirtualBox:~/auth$ ssh otusadm@192.168.57.10
otusadm@192.168.57.10's password: 
Last login: Sat Feb  3 09:32:21 2024 from 192.168.57.1
[otusadm@pam ~]$ whoami
otusadm
[otusadm@pam ~]$ exit
logout
Connection to 192.168.57.10 closed.
user@user-VirtualBox:~/auth$ 
```
Скрипт работает корректно.
