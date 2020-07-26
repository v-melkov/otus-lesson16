## Стенд для домашнего задания "Авторизация и аутентификация"

### Домашнее задание

-   Запретить всем пользователям, кроме группы admin логин в выходные (суббота и воскресенье), без учета праздников
-   дать конкретному пользователю права работать с докером и возможность рестартить докер сервис

* * *

#### Задание 1

Для начала создадим группу admin и пользователей, входящих и не входящих в группу admin:  

    groupadd admin
    useradd admin1 && echo "pass"|passwd --stdin admin1 && usermod -a -G admin admin1; \\
    useradd notadmin1 && echo "pass"|passwd --stdin notadmin1

Далее воспользуемся комбинацией из двух модулей pam_succeed_if и pam_time

Добавим правило в time.conf

    #разрешить всем любые действия в системе в любой день кроме выходных
    echo '*;*;*;!Wd0000-2400' >> /etc/security/time.conf

Добавим правила в файл /etc/pam.d/system-auth:  

    account [success=1 default=ignore] pam_succeed_if.so user ingroup admin
    account required pam_time.so

Логика работы правил:
если пользователь входит в группу admin, следующее правило (account required pam_time.so) пропускается. Иначе производится проверка на время модулем pam_time.so.

* * *

##### Проверка

Пароль **pass**  

Выполним команды:  
`sudo date +%Y%m%d -s "20200726"`  
`date`  
Вывод:

    Sun Jul 26 00:00:02 UTC 2020 (выходной)  

`su admin1` - доступ разрешён  
`su notadmin1` - доступ запрещён

`sudo date +%Y%m%d -s "20200727"`  
`date`  
Вывод:  

    Mon Jul 27 00:00:07 UTC 2020 (не выходной)  

`su admin1` - доступ разрешён  
`su notadmin1` - доступ разрешён

* * *

#### Задание 2

Установим и запустим docker:  

    dnf config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
    dnf install -y --nobest docker-ce
    systemctl enable --now docker
    systemctl status docker | head -n3

Добавим пользователя docker-user и добавим его в группу docker  
`useradd docker-user && echo "pass"|passwd --stdin docker-user && usermod -a -G docker docker-user`  

После добавления в группу docker пользователю разрешена работа с docker-контейнерами (запуск, остановка и т.д.)  
Проверка командой `sudo -u docker-user docker run hello-world`  

Перезапуск докер-сервиса требует аутентификации root.  
`sudo -u docker-user systemctl restart docker`  

    ==== AUTHENTICATING FOR org.freedesktop.systemd1.manage-units ====
    Authentication is required to restart 'docker.service'.

Добавим правило разрешающее рестарт сервиса докер пользователю docker-user:  
`cat /etc/polkit-1/rules.d/01-docker.rules`  

    //правило разрешающее перезапуск сервиса докер пользователю docker-user
    polkit.addRule(function(action, subject) {
      if (action.id == "org.freedesktop.systemd1.manage-units" && action.lookup("unit") == "docker.service" && action.lookup("verb") == "restart" && subject.user == "docker-user") {
          return polkit.Result.YES;
        }
    });

Проверим командой `sudo -u docker-user systemctl restart docker`  
Сервис перезапустился.

Задание выполнено.

### Спасибо за проверку!
