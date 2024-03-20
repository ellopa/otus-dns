## Vagrant-стенд c DNS

### Введение

DNS(Domain Name System, Служба доменных имён) - это распределенная система, для получения информации о доменах. DNS используется для сопоставления
IP-адресов и доменных имён.

Сопостовления IP-адресов и DNS-имён бывают двух видов:

- Прямое (DNS-имя в IP-адрес)
- Обратное (IP-адрес в DNS-имя)

Доменная структура DNS представляет собой древовидную иерархию, состоящую из узлов, зон, доменов, поддоменов и т.д. «Вершиной» доменной структуры является корневая зона. Корневая (root) зона обозначается точкой. Далее следуют домены первого уровня (.com, ,ru, .org и т. д.) и т д.

В DNS встречаются понятия зон и доменов:

- Зона — это любая часть дерева системы доменных имён, размещаемая как единое целое на некотором DNS-сервере.
- Домен – определенный узел, включающий в себя все подчинённые узлы.

Давайте разберем основное отличие зоны от домена. Возьмём для примера ресурс otus.ru — это может быть сразу и зона и домен, однако, при использовании зоны otus.ru мы можем сделать отдельную зону mail.otus.ru, которая будет управляться не нами. В случае домена так сделать нельзя...

FQDN (Fully Qualified Domain Name) - полностью указанное доменное имя, т.е. от корневого домена. Ключевой идентификатор FQDN - точка в конце имени. Максимальный размер FQDN — 255 байт, с ограничением в 63 байта на каждое имя домена. Пример FQDN: mail.otus.ru.

Вся информация о DNS-ресурсах хранится в ресурсных записях. Записи хранят следующие атрибуты:

- Имя (NAME) - доменное имя, к которому привязана или которому принадлежит данная ресурсная область, либо IP-адрес. При отсутствии данного поля, запись ресурса наследуется от предыдущей записи.
- TTL (время жизни в кэше) - после указанного времени запись удаляется, данное поле может не указываться в индивидуальных записях ресурсов, но тогда оно должно быть указано в начале файла зоны и будет наследоваться всеми записями.
- Класс (CLASS) - определяет тип сети (в 99% используется IN - интернет)
- Тип (TYPE) - тип записи, синтаксис и назначение записи
- Значение (DATA)

Типы рекурсивных записей:

- А (Address record) - отображают имя хоста (доменное имя) на адрес IPv4
- AAAA - отображает доменное имя на адрес IPv6
- CNAME (Canonical name reord/псевдоним) - привязка алиаса к существующему доменному имени
- MX (mail exchange) - указывает хосты для отправки почты, адресованной домену. При этом поле NAME указывает домен назначения, а поле DATA приоритет и доменное имя хоста, отвественного за приём почты. Данные вводятся через пробел
- NS (name server) - указывает на DNS-сервер, обслуживающий данный домен.
- PTR (pointer) - Отображает IP-адрес в доменное имя
- SOA (Start of Authority/начальная запись зоны) - описывает основные начальные настройки зоны.
- SRV (server selection) — указывает на сервера, обеспечивающие работу тех или иных служб в данном домене (например Jabber и Active Directory).

Для работы с DNS (как клиенту) в linux используют утилиты dig, host и nslookup

Также в Linux есть следующие реализации DNS-серверов:

- bind
- powerdns (умеет хранить зоны в БД)
- unbound (реализация bind)
- dnsmasq
- и т д.

Split DNS (split-horizon или split-brain) — это конфигурация, позволяющая отдавать разные записи зон DNS в зависимости от подсети источника запроса. Данную функцию можно реализовать как с помощью одного DNS-сервера, так и с помощью нескольких DNS-серверов…

### Цели домашнего задания

Создать домашнюю сетевую лабораторию. Изучить основы DNS, научиться работать с технологией Split-DNS в Linux-based системах.

### Описание домашнего задания

### 1. Взять стенд <https://github.com/erlong15/vagrant-bind>

- добавить еще один сервер client2
- завести в зоне dns.lab имена:
- web1 - смотрит на клиент1
- web2 смотрит на клиент2
- завести еще одну зону newdns.lab
- завести в ней запись
- www - смотрит на обоих клиентов

#### 2. Настроить split-dns

- клиент1 - видит обе зоны, но в зоне dns.lab только web1
- клиент2 видит только dns.lab

### Работа со стендом и настройка DNS

### 1. Развернуть стенд

- Скачать стенд <https://github.com/erlong15/vagrant-bind> и изучить содержимое файлов:

```bash
git clone https://github.com/erlong15/vagrant-bind.git
elena_leb@ubuntunbleb:~/DNS_DZ$ cd vagrant-bind
elena_leb@ubuntunbleb:~/DNS_DZ/vagrant-bind$ ls -l
итого 16
drwxrwxr-x 2 elena_leb elena_leb 4096 мар 14 08:17 provisioning
-rw-rw-r-- 1 elena_leb elena_leb  414 мар 14 07:38 README.md
-rw-rw-r-- 1 elena_leb elena_leb  994 мар 14 17:25 Vagrantfile

```
- В файл Vagrantfile добавить еще один сервер client2 - [Vagrantfile](Vagrantfile_1)

>- Vagranfile описывает создание 4 виртуальных машин на CentOS 7, каждой машине будет выделено по 256 МБ ОЗУ. В начале файла есть модуль, который отвечает за настройку ВМ с помощью Ansible.

 - [playbook.yml](playbook_1.yml) — это Ansible-playbook, в котором содержатся инструкции по настройке стенда
 - client-motd — файл, содержимое которого будет появляться перед пользователем, который подключился по SSH
 - named.ddns.lab и named.dns.lab — файлы описания зон ddns.lab и dns.lab соответсвенно
 - master-named.conf и slave-named.conf — конфигурационные файлы, в которых хранятся настройки DNS-сервера
 - client-resolv.conf и servers-resolv.conf — файлы, в которых содержатся IP-адреса DNS-серверов

>- **Для нормальной работы DNS-серверов, на них должно быть настроено одинаковое время.** Для того, чтобы на всех серверах было одинаковое время, необходимо настроить NTP. В CentOS по умолчанию уже есть NTP-клиент Chrony. Обычно он всегда включен и добавлен в автозагрузку. Проверить работу службы можно командой: systemctl status chronyd

>- **При варианте установки утилиты ntp, после установки необходимо:**
>-  - Остановить службу chronyd: systemctl stop chronyd
>-  - Удалить службу chronyd из автозагрузки: systemctl disable chronyd
>-  - Включить службу ntpd: systemctl start ntpd
>-  - Добавить службу ntpd в автозагрузку: systemctl enable ntpd

>- Пример данной настройки в Ansible (YAML-формат):
```yml
- name: stop and disable chronyd
  service:
    name: chronyd
    state: stopped
    enabled: false
- name: start and enable ntpd
  service:
    name: ntpd
    state: started
    enabled: true
```
>- **Альтернативный вариант — не устанавливать ntp и просто запустить службу chronyd: systemctl start chronyd**
>- Пример данной настройки в Ansible (YAML формат):

```yml
- name: install packages
  yum:
    name:
    - bind
    - bind-utils
    - vim
    state: latest
    update_cache: true

- name: start chronyd
  service:
    name: chronyd
    state: restarted
    enabled: true
```
- В данном развертывании будет установлена утилита ntp
>- **vagrant up --provision** - это команда, которая используется для настройки внутреннего окружения имеющейся виртуальной машины. Например, она позволяет добавить новые инструкции (скрипты) в ранее созданную виртуальную машину. 

- Проверить на каком адресе и порту работают DNS-сервера можно двумя способами:

- - Посмотреть с помощью команды SS: **ss -tulpn**

```bash
[vagrant@ns01 ~]$ ss -tulpn
Netid  State      Recv-Q Send-Q               Local Address:Port                              Peer Address:Port              
udp    UNCONN     0      0                    192.168.56.20:53                                           *:*                  
udp    UNCONN     0      0                                *:68                                           *:*                  
udp    UNCONN     0      0                                *:111                                          *:*                  
udp    UNCONN     0      0                    192.168.56.20:123                                          *:*                  
udp    UNCONN     0      0                        10.0.2.15:123                                          *:*                  
udp    UNCONN     0      0                        127.0.0.1:123                                          *:*                  
udp    UNCONN     0      0                                *:123                                          *:*                  
udp    UNCONN     0      0                                *:962                                          *:*                  
udp    UNCONN     0      0                            [::1]:53                                        [::]:*                  
udp    UNCONN     0      0                             [::]:111                                       [::]:*                  
udp    UNCONN     0      0      [fe80::5054:ff:fe4d:77d3]%eth0:123                                       [::]:*                  
udp    UNCONN     0      0                            [::1]:123                                       [::]:*                  
udp    UNCONN     0      0      [fe80::a00:27ff:fe0f:5ddb]%eth1:123                                       [::]:*                  
udp    UNCONN     0      0                             [::]:123                                       [::]:*                  
udp    UNCONN     0      0                             [::]:962                                       [::]:*                  
tcp    LISTEN     0      128                              *:22                                           *:*                  
tcp    LISTEN     0      128                  192.168.56.20:953                                          *:*                  
tcp    LISTEN     0      100                      127.0.0.1:25                                           *:*                  
tcp    LISTEN     0      128                              *:111                                          *:*                  
tcp    LISTEN     0      10                   192.168.56.20:53                                           *:*                  
tcp    LISTEN     0      128                           [::]:22                                        [::]:*                  
tcp    LISTEN     0      100                          [::1]:25                                        [::]:*                  
tcp    LISTEN     0      128                           [::]:111                                       [::]:*                  
tcp    LISTEN     0      10                           [::1]:53                                        [::]:* 
```
- Посмотреть информацию в настройках DNS-сервера (**/etc/named.conf**)
```
[root@ns01 ~]# cat /etc/named.conf 
options {

    // network 
	listen-on port 53 { 192.168.56.20; };
	listen-on-v6 port 53 { ::1; };
```
```
[root@ns02 ~]# cat /etc/named.conf
options {

    // network 
	listen-on port 53 { 192.168.56.21; };
	listen-on-v6 port 53 { ::1; };
```

- На основе данной информации, нужно внести корректировки в файл /etc/resolv.conf для DNS-серверов:на хосте ns01 указать nameserver 192.168.56.20, а на хосте ns02 — 192.168.56.21

>- В Ansible для этого можно воспользоваться шаблоном с Jinja. Изменим имя файла servers-resolv.conf на servers-resolv.conf.j2 и укажем там следующие условия:

```bash
domain dns.lab
search dns.lab
#Если имя сервера ns02, то указываем nameserver 192.168.56.21
{% if ansible_hostname == 'ns02' %}
nameserver 192.168.56.21
{% endif %}
#Если имя сервера ns01, то указываем nameserver 192.168.56.20
{% if ansible_hostname == 'ns01' %}
nameserver 192.168.56.20
{% endif %}
```
>- После внесения измений в файл, нужно изменить ansible-playbook - вместо модуля copy модуль template:

```yml
- name: copy resolv.conf to the servers
  template: src=servers-resolv.conf.j2 dest=/etc/resolv.conf owner=root group=root mode=0644
```
 YAML-формат:

```yml
- name: copy resolv.conf to the servers
  template:
    src: servers-resolv.conf.j2
    dest: /etc/resolv.conf
    owner: root
    group: root
    mode: 0644
```
### 2. Настройка  DNS

- **Добавление имён в зону dns.lab**
- Проверить, что зона dns.lab уже существует на DNS-серверах:

```bash
// Имя зоны
zone "dns.lab" {
    type master;
    // Тем, у кого есть ключ zonetransfer.key можно получать копию файла зоны
    allow-transfer { key "zonetransfer.key"; };
    // Файл с настройками зоны
    file "/etc/named/named.dns.lab";
};
```
- Фрагмент файла /etc/named.conf на сервере ns01:
```
// lab's zone
zone "dns.lab" {
    type master;
    allow-transfer { key "zonetransfer.key"; };
    file "/etc/named/named.dns.lab";
};
```

- Похожий фрагмент файла /etc/named.conf находится на slave-сервере ns02:
```
// lab's zone
zone "dns.lab" {
    type slave;
    masters { 192.168.56.20; };
    file "/etc/named/named.dns.lab";
};
```
- На хосте ns01 мы видим файл /etc/named/named.dns.lab с настройкой зоны:

```bash
[root@ns01 named]# cat named.dns.lab
$TTL 3600
$ORIGIN dns.lab.
@               IN      SOA     ns01.dns.lab. root.dns.lab. (
                            2711201407 ; serial
                            3600       ; refresh (1 hour)
                            600        ; retry (10 minutes)
                            86400      ; expire (1 day)
                            600        ; minimum (10 minutes)
                        )

                IN      NS      ns01.dns.lab.
                IN      NS      ns02.dns.lab.

; DNS Servers
ns01            IN      A       192.168.56.20
ns02            IN      A       192.168.56.21
```
- В этот файл добавить имена:

```bash
;Web
web1            IN      A       192.168.56.25
web2            IN      A       192.168.56.26
```
```
[root@ns01 named]# vi named.dns.lab
[root@ns01 named]# cat named.dns.lab
$TTL 3600
$ORIGIN dns.lab.
@               IN      SOA     ns01.dns.lab. root.dns.lab. (
                            2711201407 ; serial
                            3600       ; refresh (1 hour)
                            600        ; retry (10 minutes)
                            86400      ; expire (1 day)
                            600        ; minimum (10 minutes)
                        )

                IN      NS      ns01.dns.lab.
                IN      NS      ns02.dns.lab.

; DNS Servers
ns01            IN      A       192.168.56.20
ns02            IN      A       192.168.56.21
;Web
web1            IN      A       192.168.56.25
web2            IN      A       192.168.56.26
```
>- Если изменения внесены вручную, то для применения настроек нужно:
-  Перезапустить службу named: systemctl restart named
-  Изменить значение Serial (добавить +1 к числу 2711201407), изменение значения serial укажет slave-серверам на то, что были внесены изменения и что им надо обновить свои файлы с зонами.

- Выполнить проверку с клиента - обращениек  разным DNS-серверам с разными запросами:

```bash
[vagrant@client ~]$ dig @192.168.56.20 web1.dns.lab

; <<>> DiG 9.11.4-P2-RedHat-9.11.4-26.P2.el7_9.15 <<>> @192.168.56.20 web1.dns.lab
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 24205
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 2, ADDITIONAL: 3

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;web1.dns.lab.			IN	A

;; ANSWER SECTION:
web1.dns.lab.		3600	IN	A	192.168.56.25

;; AUTHORITY SECTION:
dns.lab.		3600	IN	NS	ns02.dns.lab.
dns.lab.		3600	IN	NS	ns01.dns.lab.

;; ADDITIONAL SECTION:
ns01.dns.lab.		3600	IN	A	192.168.56.20
ns02.dns.lab.		3600	IN	A	192.168.56.21

;; Query time: 0 msec
;; SERVER: 192.168.56.20#53(192.168.56.20)
;; WHEN: Sun Mar 17 12:13:57 MSK 2024
;; MSG SIZE  rcvd: 127

```
>- web1.dns.lab.		3600	IN	A	192.168.56.25

```bash
dig @192.168.56.21 web2.dns.lab
[vagrant@client ~]$ dig @192.168.56.21 web2.dns.lab

; <<>> DiG 9.11.4-P2-RedHat-9.11.4-26.P2.el7_9.15 <<>> @192.168.56.21 web2.dns.lab
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 8351
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 2, ADDITIONAL: 3

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;web2.dns.lab.			IN	A

;; ANSWER SECTION:
web2.dns.lab.		3600	IN	A	192.168.56.26

;; AUTHORITY SECTION:
dns.lab.		3600	IN	NS	ns02.dns.lab.
dns.lab.		3600	IN	NS	ns01.dns.lab.

;; ADDITIONAL SECTION:
ns01.dns.lab.		3600	IN	A	192.168.56.20
ns02.dns.lab.		3600	IN	A	192.168.56.21

;; Query time: 0 msec
;; SERVER: 192.168.56.21#53(192.168.56.21)
;; WHEN: Sun Mar 17 12:14:06 MSK 2024
;; MSG SIZE  rcvd: 127

```
>- web2.dns.lab.           3600    IN      A       192.168.50.16

- Добавление имён в зону dns.lab с помощью Ansible

>- В существующем Ansible-playbook менять ничего не потребуется, нужно добавить строки с новыми именами в файл [named.dns.lab](named.dns_2.lab) и снова запустить Ansible.

- **Создание новой зоны и добавление в неё записей**

- Для того, чтобы прописать на DNS-серверах новую зону необходимо:

- На хосте ns01 добавить зону в файл /etc/named.conf:

```bash
// lab's newdns zone
zone "newdns.lab" {
     type master;
     allow-transfer { key "zonetransfer.key"; };
     allow-update { key "zonetransfer.key"; };
     file "/etc/named/named.newdns.lab";
};
```

- На хосте ns02 также добавить зону и указать с какого сервера запрашивать информацию об этой зоне (фрагмент файла /etc/named.conf):

```bash
// lab's newdns zone
zone "newdns.lab" {
    type slave;
    masters { 192.168.56.20; };
    file "/etc/named/named.newdns.lab";
};
```
- На хосте ns01 создадим файл /etc/named/named.newdns.lab

```bash
vi /etc/named/named.newdns.lab
$TTL 3600
$ORIGIN newdns.lab.
@               IN      SOA     ns01.dns.lab. root.dns.lab. (
                            2711201407 ; serial
                            3600       ; refresh (1 hour)
                            600        ; retry (10 minutes)
                            86400      ; expire (1 day)
                            600        ; minimum (10 minutes)
                        )

                IN      NS      ns01.dns.lab.
                IN      NS      ns02.dns.lab.

; DNS Servers
ns01            IN      A       192.168.56.20
ns02            IN      A       192.168.56.21
; WWW
www            IN      A       192.168.56.25
www            IN      A       192.168.56.26
```

В конце этого файла добавим записи "www". У файла должны быть права 660, владелец — root, группа — named.
```
[root@ns01 named]# ls -l /etc/named/named.newdns.lab
-rw-rw----. 1 root named 700 Mar 17 12:49 /etc/named/named.newdns.lab
```
- После внесения данных изменений, изменяем значение serial (добавлем +1 к значению 2711201007) и перезапускаем named: systemctl restart named

- Создание новой зоны и добавление в неё записей **с помощью Ansible**. Для создания зоны и добавления в неё записей, нужно добавить зону в файл /etc/named.conf на хостах ns01 [master-named.conf](master-named_2.conf) и ns02 [slave-named.conf ](slave-named_2.conf), а также создать файл [named.newdns.lab](named.dns.lab), которые будут копироваться на сервер ns01 и ns02 через playbook.
- Добавить в модуль copy файл named.newdns.lab [playbook](playbook_2.yml)

```yml
- name: copy zones
    copy: src={{ item }} dest=/etc/named/ owner=root group=named mode=0660
    with_fileglob:
      - named.d*
      - named.newdns.lab
```
>- Соответственно файл named.newdns.lab будет скопирован на хост ns01 по адресу /etc/named/named.newdns.lab


### Настройка Split-DNS

- Прописанные зоны dns.lab и newdns.lab. 
  Необходимо:
   - client1 должен видеть запись web1.dns.lab и не видеть запись web2.dns.lab. 
   - client2 может видеть обе записи из домена dns.lab, но не должен видеть записи домена newdns.lab

- **Для настройки Split-DNS нужно:**

- Создать дополнительный файл зоны dns.lab, в котором будет прописана только одна запись: vim /etc/named/named.dns.lab.client
>- **Имя файла может отличаться от указанной зоны. У файла должны быть права 660, владелец — root, группа — named.**
```
[root@ns01 named]# touch /etc/named/named.dns.lab.client
[root@ns01 named]# chgrp named /etc/named/named.dns.lab.client
[root@ns01 named]# chmod 660 /etc/named/named.dns.lab.client
[root@ns01 named]# ls -l /etc/named/named.dns.lab.client
-rw-rw----. 1 root named 0 Mar 17 13:50 /etc/named/named.dns.lab.client
```
```bash
[root@ns01 named]# cat /etc/named/named.dns.lab.client
$TTL 3600
$ORIGIN dns.lab.
@ IN SOA ns01.dns.lab. root.dns.lab. (
2711201407 ; serial
3600 ; refresh (1 hour)
600 ; retry (10 minutes)
86400 ; expire (1 day)
600 ; minimum (10 minutes)
)
IN NS ns01.dns.lab.
IN NS ns02.dns.lab.
; DNS Servers
ns01 IN A 192.168.56.20
ns02 IN A 192.168.56.21
;Web
web1 IN A 192.168.56.25

```
- Внести изменения в файл /etc/named.conf на хостах ns01 и ns02

- Прежде всего нужно сделать access листы для хостов client и client2. Сначала сгенерируем ключи для хостов client и client2, для этого на хосте ns01 запустим утилиту tsig-keygen (ключ может генериться 5 минут и более):

```bash
[root@ns01 named]# tsig-keygen
key "tsig-key" {
	algorithm hmac-sha256;
	secret "SRr1/Sc4Vuc4Atsypblxvt1ni9Om9HQZs4O6Qr750XU=";
};
```
- После генерации, мы увидем ключ (secret) и алгоритм с помощью которого он был сгенерирован. Оба этих параметра нам потребуются в access листе.
>- Если нам потребуется, использовать другой алогоритм, то мы можем его указать как аргумент к команде, например: tsig-keygen -a hmac-md5

- Всего нам потребуется 2 таких ключа. 
```
[vagrant@client1 ~]$ tsig-keygen client1-key
key "client1-key" {
	algorithm hmac-sha256;
	secret "AsFOq4fmhUv1mUicwwU92jpBfk5sy+R05ssME+y8eY0=";
};

```
```
[vagrant@client2 ~]$ tsig-keygen client2-key
key "client2-key" {
	algorithm hmac-sha256;
	secret "rs0EjZasN9S0vJ2uwXij1yZGHn/MXmPLiKKY9swm37I=";
};
```
- После их генерации добавим блок с access листами в конец файла /etc/named.conf

```
key "client1-key" {
	algorithm hmac-sha256;
	secret "AsFOq4fmhUv1mUicwwU92jpBfk5sy+R05ssME+y8eY0=";
};

key "client2-key" {
    algorithm hmac-sha256;
    secret "rs0EjZasN9S0vJ2uwXij1yZGHn/MXmPLiKKY9swm37I=";
};
acl client1 { !key client2-key; key client1-key; 192.168.56.25; };
acl client2 { !key client1-key; key client2-key; 192.168.56.26; };
```
- В данном блоке access листов мы выделяем 2 блока:

- client1 имеет адрес 192.168.56.25, использует client1-key и не использует client2-key
- client2 имеет адрес 192ю168.56.26, использует clinet2-key и не использует client1-key

- Описание ключей и access листов будет одинаковое для master и slave сервера.

Далее нужно создать файл с настройками зоны dns.lab для client, для этого на мастер сервере создаём файл /etc/named/named.dns.lab.client и добавляем в него следующее содержимое:

```bash
$TTL 3600
$ORIGIN dns.lab.
@               IN      SOA     ns01.dns.lab. root.dns.lab. (
                            2711201407 ; serial
                            3600       ; refresh (1 hour)
                            600        ; retry (10 minutes)
                            86400      ; expire (1 day)
                            600        ; minimum (10 minutes)
                        )

                IN      NS      ns01.dns.lab.
                IN      NS      ns02.dns.lab.

; DNS Servers
ns01            IN      A       192.168.56.20
ns02            IN      A       192.168.56.21
; Web
web1            IN      A       192.168.56.25

```

>- Это почти скопированный файл зоны dns.lab, в конце которого удалена строка с записью web2. Имя зоны надо оставить такое же — dns.lab

- Далее нужно внести правки в /etc/named.conf

>- Технология Split-DNS реализуется с помощью описания представлений (view), для каждого отдельного acl. В каждое представление (view) добавляются только те зоны, которые разрешено видеть хостам, адреса которых указаны в access листе.

>- **Все ранее описанные зоны должны быть перенесены в модули view. Вне view зон быть недолжно, зона 'any' должна всегда находиться в самом низу.**

- После применения всех вышеуказанных правил на хосте ns01 мы получим следующее содержимое файла [/etc/named.conf](master-named.conf)

- Внести изменения в файл /etc/named.conf на сервере ns02. Файл будет похож на файл для ns01, только в настройках будет указание забирать информацию с сервера ns01 и указан тип slave. Итоговый файл для ns02 [/etc/named.conf](slave-named.conf)

>- Так как файлы с конфигурациями получаются достаточно большими — возрастает вероятность сделать ошибку. При их правке можно воспользоваться утилитой **named-checkconf**. Она укажет в каких строчках есть ошибки. Использование данной утилиты рекомендуется после изменения настроек на DNS-сервере.
```
named-checkconf /etc/named.conf
named-checkconf -t /var/named/chroot /etc/named.conf 
```
- После внесения данных изменений можно перезапустить (по очереди) службу named на серверах ns01 и ns02.

- Проверка работы Split-DNS с хостов client и client2. Для проверки можно использовать утилиту ping:
```bash
ping www.newdns.lab
ping web1.dns.lab
ping web2.dns.lab
```
<img src="https://github.com/ellopa/otus-dns/blob/main/scr_dns_dz_1.png" width=50% height=50%>
>- **На хосте мы видим, что client1 видит обе зоны (dns.lab и newdns.lab), однако информацию о хосте web2.dns.lab он получить не может.**

<img src="https://github.com/ellopa/otus-dns/blob/main/scr_dns_dz_2.png" width=50% height=50%>
>- **client2 видит всю зону dns.lab и не видит зону newdns.lab**

- Для того, чтобы проверить что master и slave сервера отдают одинаковую информацию, в файле /etc/resolv.conf можно удалить на время nameserver 192.168.50.20 и попробовать выполнить все те же проверки. Результат должен быть идентичный. 
<img src="https://github.com/ellopa/otus-dns/blob/main/scr_dns_dz_3.png" width=50% height=50%>

### Настройка Split-DNS c помощью Ansible

[Ansible-playbook](playbook.yml) менять ничего не потребуется. 
Нужно изменить содержимое файлов [master-named.conf](master-named.conf) и [slave-named.conf](slave-named.conf), а также добавить файл [named.dns.lab.client](named.dns.lab.client).



>- После перезапуска виртуалок стенда, файл /etc/resolv.conf перезаписывался
```
cat /etc/resolv.conf
# Generated by NetworkManager
nameserver 10.0.2.3
```
>- Запрет изменения файла /etc/resolv.conf
```
[root@client1 ~]# chattr +i /etc/resolv.conf
[root@client1 ~]# ls -la /etc/resolv.conf
-rw-r--r--. 1 root root 80 Mar 20 11:31 /etc/resolv.conf
[root@client1 ~]# lsattr /etc/resolv.conf
----i----------- /etc/resolv.conf
```
>- Запрет изменения файла /etc/resolv.conf добавлено в playbook для клиентов
```
  - name: makefile immutable 
    command: chattr +i /etc/resolv.conf 
```
### Статьи 

- Статья o DNS (общая информация) - <https://ru.wikipedia.org/wiki/DNS>
- Статья «DNS сервер BIND (теория)» - <https://habr.com/ru/post/137587/>
- Статья «Настройка Split DNS на одном сервере Bind» - <https://www.dmosk.ru/miniinstruktions.php?mini=split-dns-bind>
- Статья «Split DNS: заставим BIND работать на два фронта!» - <http://samag.ru/archive/article/771>
- Статья «DNS BIND Zone Transfers and Updates» - <https://www.zytrax.com/books/dns/ch7/xfer.html#:~:text=also%2Dnotify%20Statement%20(Pre%20BIND9.9)&text=also%2Dnotify%20defines%20a%20list,NS%20records%20for%20the%20zone>
- Статья - «CentOS 7 resolv.conf make changes permanent» - <https://serverok.in/centos-7-resolv-conf-permanent?ysclid=ltzjlndw6n80752007>
