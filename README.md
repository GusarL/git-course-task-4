# Последовательность действий:

1. Копирую чистый репозиторий, без рабочей папки, на локальную машину
   git clone --bare https://github.com/MaksymSemenykhin/git-course-task.git
2. Захожу на сервер
   ```
   ssh lhus@devopstest.pp.ua.com
   ```
3. Делаю папку git в opt на сервере
   ```
   cd ..
   cd ..
   cd opt
   mkdir git 
   ```
4. Делаю доступ на запись 
   ```
   sudo chmod -R 700 /opt 
   ```
5. В папке, куда склонился репозиторий на локальной машине, копирую склоненный репозиторий на сервер
   ```
   scp -r -i git-course-task.git lhus@devopstest.pp.ua:/opt/git
   ```
6. На сервере в папке opt/git
  ```
  mkdir git-course-task.git
  git-course-task.git
  git init --bare --shared
  ``` 
7. Добавила ключ god.pub, noob.pub, pro.pub на сервер
   ```
   ssh lhus@devopstest.pp.ua:/opt/git
   cd .ssh/authorized_keys  
   ```
   скопировала содержимое файлов god.pub, noob.pub, pro.pub и сохранила
8. Добавила пользователей god, noob и pro
   ```
   sudo adduser <username>
   su <username>
   cd
   mkdir .ssh && chmod 700 .ssh
   touch .ssh/authorized_keys && chmod 600 .ssh/authorized_keys 
   пароль <password> 
   ```
   Пароли: noob 2222
           pro 3333
           god 4444

9. Добавила ключи god.pub, noob.pub, pro.pub для пользователей god, noob и pro в .ssh/authorized_keys.                   

10. Дала права записи в папку var/www/task4
    ```
    sudo chown -R `whoami`:`id -gn` /var/www/task4
    ```
11. Перехожу в opt/git/git-course-task.git
    ```
    nano .git/hooks/update
    ```
12. Пишу bash-script, который как только сервер получить пуш, ищет изменения в master или dev, 
    если находит, обновляет папку /var/www/task4/master или /var/www/task4/dev соответственно 
    ```
    #!/bin/bash
    if [[ $ref =~ .*/master$ ]];
    then
        echo "Master ref received.  Deploying master branch to production..."
    
        git --work-tree=/var/www/task4/master --git-dir=/opt/git/git-course-task.git checkout -f
    elif [[ $ref =~ .*/dev$ ]];
    then
    echo "Dev ref received.  Deploying dev branch to production..."
        git --work-tree=/var/www/task4/dev --git-dir=/opt/git/git-course-task.git checkout -f
    else
        echo "Ref $ref successfully received.  Doing nothing: only the master or dev branch may be deployed on this server."
    fi
    done

    ```
13. Делаю файл исполняемым
    ```
    chmod +x hooks/update
    ```
14. Добавляю удалённые вертки production и current 
    ```
     git remote add production git@devopstest.pp.ua:/opt/git/git-course-task.git
     git remote add current git@devopstest.pp.ua:/opt/git/git-course-task.git
     ```
15. Меняю ссылку на удалённой ветке origin
    ```
     git remote set-url origin git@devopstest.pp.ua:/opt/git/git-course-task.git
    ``` 
16. Создаю группы master и dev и добавляю в них аользователей:
    ```
    sudo groupadd master
    sudo usermod -a -G master pro
    sudo usermod -a -G master god
    sudo groupadd dev
    sudo usermod -a -G dev noob
    sudo usermod -a -G dev pro
    sudo usermod -a -G dev god
    ``` 
17. Меняю собственника групп  
    ```
     sudo chgrp -R master /var/www/task4/master
     sudo chgrp -R dev /var/www/task4/dev
     ```   
18. Добавляю права на запись
    ```
     sudo chmod -R g+w /var/www/task4/master
     sudo chmod -R g+w /var/www/task4/dev
    ```  
