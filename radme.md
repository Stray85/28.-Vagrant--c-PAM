### Vagrant-стенд c PAM  
## Цель домашнего задания  
Научиться создавать пользователей и добавлять им ограничения  
  
# Описание домашнего задания  
Запретить всем пользователям, кроме группы admin, логин в выходные (суббота и воскресенье), без учета праздников  
  
Создадим скрипт login.sh:   
```bash
#!/bin/bash  
#Первое условие: если день недели суббота или воскресенье  
if [ $(date +%a) = "Sat" ] || [ $(date +%a) = "Sun" ]; then  
 #Второе условие: входит ли пользователь в группу admin  
 if getent group admin | grep -qw "$PAM_USER"; then  
        #Если пользователь входит в группу admin, то он может подключиться  
        exit 0  
      else  
        #Иначе ошибка (не сможет подключиться)  
        exit 1  
    fi  
  #Если день не выходной, то подключиться может любой пользователь  
  else  
    exit 0  
fi  
```
  
В скрипте подписаны все условия. Скрипт работает по принципу:   
Если сегодня суббота или воскресенье, то нужно проверить, входит ли пользователь в группу admin, если не входит — то подключение запрещено. При любых других вариантах подключение разрешено.   
  
  
  
Разрешаем подключение пользователей по SSH с использованием пароля  
```sed -i 's/^PasswordAuthentication.*$/PasswordAuthentication yes/' /etc/ssh/sshd_config```  
Перезапуск службы SSHD  
 ```systemctl restart sshd.service```  
Скопируем файл-скрипт login.sh в /usr/local/bin/  
 ```cp /vagrant/login.sh /usr/local/bin/```  
Добавим права на исполнение файла:  
 ```chmod +x /usr/local/bin/login.sh```  
Создаём пользователя otusadm и otus:  
 ```useradd otusadm && sudo useradd otus```  
Создаём пользователям пароли:  
 ```echo "Otus2022!" | sudo passwd --stdin otusadm && echo "Otus2022!" | sudo passwd --stdin otus```  
Для примера мы указываем одинаковые пароли для пользователя otus и otusadm  
Создаём группу admin:  
 ```sudo groupadd -f admin```  
Добавляем пользователей vagrant,root и otusadm в группу admin:  
 ```usermod otusadm -a -G admin && usermod root -a -G admin && usermod vagrant -a -G admin```  
Укажем в файле /etc/pam.d/sshd модуль pam_exec и наш скрипт:  
 ```sed -i '/pam_nologin\.so$/a account    required     pam_exec.so \/usr\/local\/bin\/login\.sh' /etc/pam.d/sshd```
