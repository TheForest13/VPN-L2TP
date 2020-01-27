# VPN-L2TP

!!! Внимательно смотрите, какие файлы вам нужны !!!

В данной статье будет описана настройка L2TP VPN-тунеля между двумя машинами:

    VPN-сервер 10.10.170.1
    VPN-клиент 10.10.170.10

Установка

В качестве клиента и cервера мы будем успользовать xl2tpd. Установим необходимые пакеты:

    apt-get install l2tpd ppp

Настройка Сервера

Внесём изменения в конфигурационный файл xl2tpd (/etc/xl2tpd/xl2tpd.conf).
Общие настройки находяися в секции global:

    [global]
    
    port = 1701 ; Сервер будет слушать 1701 порт

    access control = yes ; Проверяет соответсвие IP настройкам доступа

    Настройки сервера находятся в секции lns:

    [lns servername] ; Название сервера

    ip range = 192.168.173.2-192.168.173.20 ; Диапазон выдаваемых IP адресов

    lac = 10.10.170.0 - 10.10.170.255 ; Диапазон IP адресов, которые могут присоединяться к этому серверу

    local ip = 192.168.173.1 ; IP адрес сервера в VPN сети

    require chap = yes ; Требовать у клиентов CHAP аутентификацию

    refuse pap = yes ; Не разрешать клиентам PAP аутентификацию

    require authentication = yes ; Требовать аутентификацию клиентов

    name = server ; Передавать клиентам имя сервера

    pppoptfile = /etc/ppp/options.xl2tpd; Файл с опциями ppp

В файле /etc/ppp/options.xl2tpd можно указать необходимые опции ppp, файл должен существовать.
В файле /etc/ppp/chap-secrets указываются аутентификационные данные пользователей для CHAP аутентификации:

    client server Pa$$word *

   client - логин
   server - указывается имя сервера к которому можно подсоединиться (параметр name), можно заменить на *
   Pa$$word - пароль клиента
   * - Разрешает соединения с любых IP

Добавляем сервис в автозапуск и запускаем его:

    systemctl enable xl2tpd
    systemctl start xl2tpd

Настройка Клиента

Настройка клиента производится в том же файле (/etc/xl2tpd/xl2tpd.conf), но в секции lac:

    [lac client] ; Название клиента

    lns = 10.10.170.1 ; Указывается адрес сервера к которому подсоединямся

    redial = yes ; Перподключаться при потере соединения

    redial timeout = 15 ; Сколько ждать между попытками соединиться

    require chap = yes ; Использовать CHAP аутентификацию

    refuse pap = yes ; Не использовать PAP аутентификацию
    
    require authentication = yes ; Требовать аутентификацию

    name = client ; Указываем имя с которым подсоединяемся

    pppoptfile = /etc/ppp/options.l2tpd ; Файл с опциями ppp

    autodial = yes ; Автоматически устанавливать связь при старте сервиса

Далее редактируем /etc/ppp/chap-secrets указывая сервер, имя и пароль:

    client server Pa$$word *

В файле /etc/ppp/options.l2tpd указываем опцию noauth:

    noauth

Пробуем подключиться к серверу:

     xl2tpd -D

Если подключение прошло успешно то добавляем сервис в автозапуск и запускаем его:

     systemctl enable xl2tpd
     systemctl start xl2tpd

В системе должен появиться интерфейс ppp0:

     ip a | grep ppp0
     
    74: ppp0: <POINTOPOINT,MULTICAST,NOARP,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UNKNOWN group default qlen 3
    inet 192.168.173.3 peer 192.168.173.1/32 scope global ppp0
