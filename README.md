# LDAP

-------------------------------------------------------------------

Цель домашнего задания:

Научиться настраивать LDAP-сервер и подключать к нему LDAP-клиентов

-------------------------------------------------------------------

Описание домашнего задания:

1) Установить FreeIPA

2) Написать Ansible-playbook для конфигурации клиента

-------------------------------------------------------------------

Инструкция по выполнению домашнего задания:

Создаем 3 виртуальных машины с ОС CentOS 8:

`vagrant up`

1) Настройка FreeIPA-сервер ipa.otus.lan. Установим часовой пояс:

`timedatectl set-timezone Europe/Moscow`

- Установим утилиту chrony:

`yum install -y chrony`

`systemctl enable chronyd --now`

- Выключаем Firewall:

`systemctl stop firewalld`

`systemctl disable firewalld`

- Остановим Selinux:

`setenforce 0`

- Поменяем в файле /etc/selinux/config, параметр Selinux на disabled

- Для дальнейшей настройки FreeIPA нам потребуется, чтобы DNS-сервер хранил запись о нашем LDAP-сервере. В рамках данной лабораторной работы мы не будем настраивать отдельный DNS-сервер и просто добавим запись в файл /etc/hosts:

`192.168.56.10 ipa.otus.lan ipa`

- Установим модуль DL1:

`yum install -y @idm:DL1`

- Установим FreeIPA-сервер:

`yum install -y ipa-server`

- Запустим скрипт установки:

`ipa-server-install`

- Далее, нам потребуется указать параметры нашего LDAP-сервера, после ввода каждого параметра нажимаем Enter, если нас устраивает параметр, указанный в квадратных скобках, то можно сразу нажимать Enter:

`Do you want to configure integrated DNS (BIND)? [no]: no`

`Server host name [ipa.otus.lan]: <Нажимаем Enter>`

`Please confirm the domain name [otus.lan]: <Нажимем Enter>`

`Please provide a realm name [OTUS.LAN]: <Нажимаем Enter>`

`Directory Manager password: <Указываем пароль минимум 8 символов>`

`Password (confirm): <Дублируем указанный пароль>`

`IPA admin password: <Указываем пароль минимум 8 символов>`

`Password (confirm): <Дублируем указанный пароль>`

`NetBIOS domain name [OTUS]: <Нажимаем Enter>`

`Do you want to configure chrony with NTP server or pool address? [no]: no`

`The IPA Master Server will be configured with:`

`Hostname:       ipa.otus.lan`

`IP address(es): 192.168.57.10`

`Domain name:    otus.lan`

`Realm name:     OTUS.LAN`

`The CA will be configured with:`

`Subject DN:   CN=Certificate Authority,O=OTUS.LAN`

`Subject base: O=OTUS.LAN`

`Chaining:     self-signed`

`Проверяем параметры, если всё устраивает, то нажимаем yes`

`Continue to configure the system with these values? [no]: yes`

- Если получаем ошибку `IPv6 stack is enabled in the kernel but there is no interface that has ::1 address assigned.`, нужно активировать ipv6 на lo0 интерфейсе:

`vim /etc/sysctl.conf`

`net.ipv6.conf.lo.disable_ipv6 = 0`

`sysctl -p`

- Далее начнется процесс установки. Процесс установки занимает примерно 5 минут (иногда время может быть другим). Если мастер успешно выполнит настройку FreeIPA то в конце мы получим сообщение: `The ipa-server-install command was successful.`

- При вводе параметров установки мы вводили 2 пароля:

`Directory Manager password` — это пароль администратора сервера каталогов, У этого пользователя есть полный доступ к каталогу.

`IPA admin password` — пароль от пользователя FreeIPA admin

После успешной установки FreeIPA, проверим, что сервер Kerberos может выдать нам билет:

image2

- Для удаление полученного билета:

`kdestroy`

`klist`

`klist: Credentials cache 'KCM:0' not found`

- Мы можем зайти в Web-интерфейс нашего FreeIPA-сервера, для этого на нашей хостой машине нужно прописать следующую строку в файле Hosts `192.168.56.10 ipa.otus.lan`

2) Конфигурация клиентов

- Настройка клиента похожа на настройку сервера. На хосте также нужно:

`Настроить синхронизацию времени и часовой пояс`

`Настроить (или отключить) firewall`

`Настроить (или отключить) SElinux`

`В файле hosts должна быть указана запись с FreeIPA-сервером и хостом`

- Хостов, которые требуется добавить к серверу может быть много, для упрощения нашей работы настройки будут выполняться с помощью Ansible.

3) Проверка работы LDAP

- На сервере FreeIPA создадим пользователя и попробуем залогиниться клиенту.

- Авторизируемся на сервере:

`kinit admin`

Создадим пользователя otus-user:

image3

- На хосте client1 или client2 выполним команду:

image4

- Система запросит у нас пароль и попросит ввести новый пароль.

- На этом процесс добавления хостов к FreeIPA-серверу завершен.
