---
permalink: /cisco-ios/
title: Cisco IOS. Знакомимся ближе.
comments: true
share: true
---

> Статья 2013 года, впервые [опубликована на сайте учебного центра "Сетевые Технологии"](http://nt.ua/aboutcenter/articles/Pages/samoilenko_cisco_ios_2013.aspx). Продублирована тут для сохранения статьи (ссылка на уц не работает).

 
Это первая статья из серии, в которой будут рассматриваться различные полезные, на мой взгляд, возможности IOS (команды, функции и др). В этой серии каждая отдельная функция или механизм не будут объясняться подробно, будет дан только обзор возможностей и несколько примеров использования. Многие из них, вероятно, будут вам знакомы, но, надеюсь, что в каждой статье вы найдете что-то новое и полезное.

 
В этой статье будут рассмотрены базовые команды IOS, которые могут быть полезны всем, независимо от того, какие функции настроены на оборудовании.
 
## Информация на сайте Cisco
 
По настройке оборудования Cisco достаточно много документации, как на официальном сайте, так и на других. Но бывает, что в поисках информации о конкретной команде или функции, приходится просматривать много сайтов и форумов, пока получается добраться до нужной информации. Возможно, небольшие подсказки помогут вам сократить время в следующий раз.
 
### Cisco Feature Navigator
 
Возможности, которые будут рассматриваться в этой статье и следующих, или те, которые интересуют вас, могут быть доступны не во всех версиях IOS. Для того чтобы проверить доступно ли это (или любая другая функция) в конкретной модели оборудования или версии IOS, можно воспользоваться [Cisco Feature Navigator (CFN)](http://tools.cisco.com/ITDIT/CFN/).
 
CFN позволяет подходить к поиску с разных сторон. Вы можете найти в каких версиях IOS и для каких платформ доступна функция, или какие функции доступны в определенном IOS. Интерфейс CFN достаточно прост и понятен, так что не нуждается в особых объяснениях.
 
### Документация на cisco.com
 
Документация на сайте cisco.com разбита на несколько разделов. Например: настройка, поиск неисправностей, дизайн. Начать знакомство с классификацией документов можно с [общей страницы](http://www.cisco.com/cisco/web/psa/default.html). Далее, можно выбрать технологию или тип устройства, для которого нужна документация.
 
Одни из основных документов, которые могут быть полезны при настройке оборудования – это command reference и configuration guide.
 
* Command reference – это документы на сайте Cisco, которые описывают команды. В них описан синтаксис команды, в каких случаях и для чего она используется, примеры использования, в какой версии IOS она появилась. Документы разделены по версиям IOS, так что лучше сразу искать документ соответствующий вашей версии. Пример описания [команды shutdown в command reference](http://www.cisco.com/en/US/docs/switches/lan/catalyst3750/software/release/12.2_55_se/commmand/reference/cli3.html#wp1944793).
 
* Configuration guide – документы на сайте Cisco, в которых описываются различные функции, протоколы и т.п. Есть и теоретические сведения, команды и примеры настройки. Документы разделены по тематике и версиям IOS, так что лучше сразу искать документ соответствующий вашей версии. [Configuration guide по настройке NAT](http://www.cisco.com/en/US/docs/ios-xml/ios/ipaddr_nat/configuration/12-4/nat-12-4-book.html).
 
## Полезные команды
 
В этом разделе рассматриваются несколько простых, но полезных в работе с оборудованием, команд.
 
### reload
 
При настройке удаленного оборудования, можно допустить ошибку в настройках, из-за которой будет потерян доступ к оборудованию. Для того чтобы подстраховаться от подобных случаев, можно воспользоваться запланированной перезагрузкой.
 
Если Вы планируете изменять настройки, которые могут привести к потери связи, то можно заранее указать, например, что маршрутизатор должен быть перезагружен через 20 минут:
```
R1# reload in 20
```
 
В таком случае, если в конфигурации была допущена ошибка, то при перезагрузке маршрутизатор вернется к исходной рабочей конфигурации.
 
Если изменения были применены и никаких проблем не возникло, то можно отменить запланированную перезагрузку: 
```
R1# reload cancel
```
 
Аналогичным образом можно указывать точное время, когда маршрутизатор должен быть перезагружен:
```
R1# reload at 20:35
```
 
## show processes
 
Во время обычной работы с оборудованием или во время поиска неисправностей, может понадобиться информация о том, какие процессы работают на маршрутизаторе, какие из них больше всего загружают процессор.
 
Команда show processes отображает информацию об активных процессах. Параметры команды позволяют отобразить информацию о статистике загрузки процессора, объеме используемой памяти, и даже представить информацию в виде графиков. Далее несколько примеров использования команды.
 
Отображение отсортированного списка процессов:
```
R1# sh processes cpu sorted
CPU utilization for five seconds: 0%/0%; one minute: 0%; five minutes: 0%
 PID Runtime(ms)     Invoked      uSecs   5Sec   1Min   5Min TTY Process
   3         708       11292         62  5.59%  3.45%  0.99%   0 Exec
 112      110826     3142856         35  4.15%  2.31%  0.62%   0 IP Input
 126     4770037   293797680         16  1.19%  0.95%  0.80%   0 IP SLAs XOS Even
  82     3767733   253040524         14  1.03%  0.66%  0.65%   0 FastEthernet Msec Ti
 252      911780    72797137         12  0.15%  0.09%  0.10%   0 MMON MENG
 108     1399765    66059754         21  0.15%  0.10%  0.10%   0 IPAM Manager
  44       81822     1645819         49  0.07%  0.00%  0.00%   0 Net Background
   7        5836       38826        150  0.00%  0.00%  0.00%   0 Pool Manager
   8           0           1          0  0.00%  0.00%  0.00%   0 DiscardQ Backgro
   9           0           2          0  0.00%  0.00%  0.00%   0 Timers
  11           0           1          0  0.00%  0.00%  0.00%   0 OIR Handler
```
 
Отображение списка процессов отсортированного по загрузке за последние 5 минут:
```
R1# sh processes cpu sorted 5min
CPU utilization for five seconds: 0%/0%; one minute: 0%; five minutes: 0%
 PID Runtime(ms)     Invoked      uSecs   5Sec   1Min   5Min TTY Process
   3         708       11299         62  0.23%  2.68%  0.98%   0 Exec
 126     4770065   293799841         16  0.95%  0.92%  0.80%   0 IP SLAs XOS Even
  82     3767765   253042444         14  0.71%  0.63%  0.64%   0 FastEthernet Msec Ti
 112      110826     3142879         35  0.00%  1.74%  0.60%   0 IP Input
 252      911788    72797633         12  0.00%  0.08%  0.10%   0 MMON MENG
 108     1399773    66060202         21  0.07%  0.09%  0.10%   0 IPAM Manager
   6      740846      392936       1885  0.07%  0.07%  0.06%   0 Check heaps
  81      164293    12959263         12  0.07%  0.01%  0.00%   0 FastEthernet Timer C
 155       37609     2324186         16  0.00%  0.00%  0.00%   0 bsm_xmt_proc
  26       54918     2426825         22  0.00%  0.00%  0.00%   0 ARP Background
  11           0           1          0  0.00%  0.00%  0.00%   0 OIR Handler
```
 
Графическое представление загрузки процессора:
```
R1# sh processes cpu history
 
    2121121112121112121 11111222222122 12211121119112121 12221
    1926405121716641818 76211100148411 70088401221831611470011
100
 90                                              *
 80                                              *
 70                                              *
 60                                              *
 50                                              *
 40                                              *
 30                               *              *
 20 **** **  ***** **** **   ****** ** ***** *   ** ***  ****
 10 ******************* **********#*** **********#****** *****
   0....5....1....1....2....2....3....3....4....4....5....5....
             0    5    0    5    0    5    0    5    0    5
               CPU% per minute (last 60 minutes)
              * = maximum CPU%   # = average CPU%
```
 
Отображение списка процессов отсортированных по объему используемой памяти:
```
R1# sh processes memory sorted
Processor Pool Total:  132000208 Used:   63513008 Free:   68487200
 
 PID TTY  Allocated      Freed    Holding    Getbufs    Retbufs Process
   0   0   83612752   37896152   40129488          0          0 *Init*         
   0   0   50331160   38366960   11307640    1709135          0 *Dead*         
 300   0    1372496      72104    1313456          0          0 EEM Server     
   1   0    3276096    2308008     981152          0          0 Chunk Manager  
 245   0     975432          0     922904          0          0 l4f mgt task   
   0   0          0          0     460536          0          0 *MallocLite*   
 287   0     352512       4992     364584     100548          0 EEM ED Syslog  
  37   0     260064       1664     271360          0          0 RF SCTPthread  
 247   0     254152          0     271216          0          0 QOS_MODULE_MAIN
   3   0    1292184    1134328     227912      22680          0 Exec           
```
 
Подробнее о команде show processes, ее параметрах и описание значений вывода команд.
 

### interface range и interface range macro 
 
При работе с коммутаторами, часто возникает задача выполнить одинаковое действие с группой портов. Для этого используется команда interface range:
```
sw1(config)# interface range fa0/1 - 5
sw1(config-if-range)# no shutdown
```
 
В команде могут быть перечислены не только порты идущие подряд:
```
sw1(config)# interface range fa0/1 - 3, fa0/6 - 8, gi0/1
sw1(config-if-range)# no shutdown
```
 
Кроме того, если какие-то диапазоны портов используются постоянно, удобно создать для этого диапазона имя (в этом примере имя диапазона "HOSTS") с помощью команды define interface-range: 
```
sw4(config)# define interface-range HOSTS fa0/1 - 3, fa0/6 - 8, gi0/1
```
 
Тогда, при дальнейших настройках этого диапазона, порты можно не перечислять, а переходить в режим настройки портов по имени:
```
sw4(config)# int range macro HOSTS
sw4(config-if-range)# no shutdown
```
 
> Примечание: Не путайте задание имени диапазону со smart macros (будут рассмотрены в следующей статье). Эти функции могут использоваться вместе.
 
### default interface
 
Иногда возникает необходимость обнулить настройки интерфейса. Но, если к интерфейсу применено большое количество команд, то удалять их может быть достаточно неудобно. Например, интерфейс настроен так:
```
interface FastEthernet0/1
 switchport access vlan 10
 switchport mode access
 switchport port-security
 switchport port-security violation restrict
 spanning-tree portfast
 spanning-tree bpduguard enable
```
 
Для того чтобы обнулить его настройки, можно воспользоваться командой default interface:
```
sw4(config)# default interface fa0/1
```
 
Эту команду можно выполнять и с диапазоном интерфейсов или используя заданное ранее имя для диапазона интерфейсов: 
```
sw1(config)# default interface range fa0/5, fa0/8 
sw1(config)# default interface range macro HOSTS
```
 
### show control-plane host open-ports
 
Команда show control-plane host open-ports показывает какие TCP и UDP-порты открыты на маршрутизаторе. У нее есть некоторые ограничения: не все сервисы, использующие UDP, отображаются (RIP), и не отображаются протоколы, которые не используют TCP и UDP (например, OSPF, EIGRP).
 
Пример вывода команды:
```
R1# show control-plane host open-ports
Active internet connections (servers and established)
Prot      Local Address           Foreign Address           Service     State
 tcp               *:22                       *:0        SSH-Server    LISTEN
 tcp               *:23                       *:0            Telnet    LISTEN
 tcp               *:80                       *:0         HTTP CORE    LISTEN
 tcp              *:179        192.168.12.2:53157               BGP  ESTABLIS
 tcp               *:22        192.168.12.2:24597        SSH-Server  ESTABLIS
 tcp              *:179                       *:0               BGP    LISTEN
 udp            *:53738          192.168.12.2:514            Syslog  ESTABLIS
```
 
## Фильтрация и перенаправление вывода командной строки
 
Как правило, со временем конфигурация разрастается и становится все сложнее просматривать ее полностью, при поиске каких-то определенных настроек. Упростить и ускорить просмотр нужных команд может фильтрация вывода.
 
Вывод любой команды show или more может быть отфильтрован или перенаправлен. Доступные параметры представлены в таблице: 

| Команда | Описание           |
| ------------- |-------------|
|include | Показать строки, в которых встретилось выражение|
|begin|Показать вывод начиная с первой строки, в которой встретилось выражение|
|section|Показывает раздел вывода, если с заданным выражением есть совпадение в заголовке раздела. Иначе, работает как include|
|exclude |Исключить строки, в которых встретилось выражение|
|redirect|Перенаправить вывод в файл|
|tee|Перенаправить вывод в файл и показать в командной строке|
|append|Добавить вывод к существующему файлу|
|count|Количество строк, которые совпадают с выражением|
 
Для того чтобы воспользоваться указанными параметрами, надо после команды show или more поставить pipe `|`,
 а затем указать параметр и ключевые слова или выражение. Далее все примеры будут показаны на командах show. Примеры использования команды more приведены ниже.
 
Команда more позволяет просматривать файлы. Например:
```
R1# more system:default-running-config
!
! Last configuration change at 15:47:50 UTC Thu Jul 25 2013
version 15.2
parser cache
parser config partition
…
 
R1# more flash:backup_ipsec
!
! Last configuration change at 12:31:49 UTC Thu Jul 25 2013
version 15.2
service timestamps debug datetime msec
service timestamps log datetime msec
no service password-encryption
!
hostname R1
!    
```
 
### Перенаправление вывода командной строки
 
Перенаправить вывод команды sh run в файл R1_config.cfg:
```
R1# show run | redirect tftp://10.0.1.1/R1_config.cfg
```
 
Перенаправить вывод команды sh ip route в файл R1_routing.cfg и показать вывод к командной строке:
```
R1# show ip route | tee tftp://10.0.1.1/R1_routing.cfg    
```
 
Добавить вывод команды sh ip int br к файлу R1_routing.cfg:
```
R1# show ip int br | append tftp://10.0.1.1/R1_routing.cfg
```
 
## Фильтрация вывода команд show
 
### include
 
Показать строки конфигурационного файла, в которых встречается "ip nat"
```
R1#sh run | i ip nat
 ip nat inside
 ip nat outside
 ip nat outside
ip nat inside source route-map NAT_ISP_R2 interface FastEthernet0/2 overload
ip nat inside source route-map NAT_ISP_R3 interface FastEthernet0/3 overload
 action 2 cli command "clear ip nat trans *"
```
 
Если нас интересуют только строки, которые начинаются на "ip nat":
```
R1#sh run | i ^ip nat
ip nat inside source route-map NAT_ISP_R2 interface FastEthernet0/2 overload
ip nat inside source route-map NAT_ISP_R3 interface FastEthernet0/3 overload
```
 
Если нужно вывести строки, которые начинаются на "ip nat" ИЛИ " ip nat" (следующие | в строке используются как ИЛИ):
```
R1#sh run | i ^ip nat|^_ip nat
 ip nat inside
 ip nat outside
 ip nat outside
ip nat inside source route-map NAT_ISP_R2 interface FastEthernet0/2 overload
ip nat inside source route-map NAT_ISP_R3 interface FastEthernet0/3 overload
```
 
Вариант этого же выражения (чтобы ввести знак вопроса надо набрать комбинацию crtl-v, а затем ?): 
```
R1#sh run | i ^_?ip nat|^interface
```
 
Добавить к выводу интерфейсы, к которым применены команды ip nat inside и ip nat outside:
```
R1#sh run | i ^ip nat|^_ip nat|^interface
interface FastEthernet0/0
 ip nat inside
interface FastEthernet0/1
interface FastEthernet0/2
 ip nat outside
interface FastEthernet0/3
 ip nat outside
ip nat inside source route-map NAT_ISP_R2 interface FastEthernet0/2 overload
ip nat inside source route-map NAT_ISP_R3 interface FastEthernet0/3 overload
```
 
### exclude
 
Исключить из вывода интерфейсы на которых не назначены IP-адреса:
```
R1#sh ip int br | e unass
Interface             IP-Address      OK? Method Status            Protocol
Ethernet0/0           10.10.10.1      YES TFTP   up                up     
Ethernet0/2           192.168.12.1    YES TFTP   up                up     
Ethernet0/3           192.168.13.1    YES TFTP   up                up     
NVI0                  10.10.10.1      YES unset  up                up     
```
 
Показать sh run без знаков восклицания:
```
R1#sh run | e !                  
Building configuration...
 
Current configuration : 3081 bytes
version 15.2
service timestamps debug datetime msec
service timestamps log datetime msec
no service password-encryption
hostname R1
boot-start-marker
boot-end-marker
no aaa new-model
```
 
### section
 
Очень удобный фильтр, который, к сожалению, есть только на маршрутизаторах (и только некоторых коммутаторах). С помощью него можно отфильтровать раздел конфигурации.
 
Например, показать настройки протоколов маршрутизации:
```
R2#sh run | s router
router eigrp 1
 network 0.0.0.0
 network 192.168.24.0
 redistribute ospf 1 metric 4000 3000 255 1 1500
 redistribute static metric 30000 200 255 1 1500
router ospf 1
 redistribute eigrp 1 subnets
 network 192.168.12.0 0.0.0.255 area 0
 default-information originate
```
 
Существующие route-map
```
R1#sh run | s ^route-map
route-map PBR_RULES permit 10
 match ip address HTTP
 set ip next-hop verify-availability 101.0.0.1 1 track 101
route-map PBR_RULES permit 20
 match ip address LAN2
 set ip next-hop verify-availability 102.0.0.1 1 track 102
route-map NAT_ISP_R3 permit 10
 match ip address NAT
 match interface Ethernet0/3
route-map NAT_ISP_R2 permit 10
 match ip address NAT
 match interface Ethernet0/2
```
 
### begin
 
С помощью параметра begin можно показать конфигурацию начиная с какой-то строки. Для коммутаторов это может использоваться и как альтернатива параметру section.
 
Показать конфигурацию начиная со строки, в которой встречается "access"
```
R1#sh run | b access  
ip access-list standard NAT
 permit 10.1.1.0 0.0.0.255
 permit 10.1.2.0 0.0.0.255
 permit 10.10.10.0 0.0.0.255
!
ip sla 13
 icmp-echo 10.1.3.100 source-interface Ethernet0/2
 threshold 200
 timeout 200
 frequency 3
ip sla schedule 13 life forever start-time now
```
 
### Фильтрация и нумерация строк конфигурации
 
Иногда может быть удобно использовать отображение номеров строк при фильтрации вывода команд. Например, если вы забыли точное название route-map, конфигурацию которой хотите посмотреть, то сначала можно дать такую команду:
```
R1#sh run linenum | i route-map
   116 : ip nat inside source route-map NAT_ISP_R2 interface Ethernet0/2 overload
   117 : ip nat inside source route-map NAT_ISP_R3 interface Ethernet0/3 overload
   143 : route-map PBR_RULES permit 10
   147 : route-map PBR_RULES permit 20
   151 : route-map NAT_ISP_R3 permit 10
   155 : route-map NAT_ISP_R2 permit 10
```
 
Теперь, когда понятно название route-map, можно, чтобы не писать его (особенно если название длинное), отфильтровать конфигурацию по номеру строки: 
```
R1#sh run linenum | b 151     
   151 : route-map NAT_ISP_R3 permit 10
   152 :  match ip address NAT
   153 :  match interface Ethernet0/3
   154 : !
   155 : route-map NAT_ISP_R2 permit 10
   156 :  match ip address NAT
   157 :  match interface Ethernet0/2
   158 : !
   159 : !
```
 
### Поиск из поля "--More--"
 
В IOS есть еще один способ фильтрации, который по действию аналогичен параметру begin. 
 
Если вывод команды достаточно длинный, то отображается только первая часть, а затем идет поле: 
```
 --More-- 
```
 
Если теперь набрать /выражение, то дальнейший вывод отфильтруется и ввод команды будет продолжен начиная со строки в которой встречается "выражение".
 
Просмотр команды sh ip interface, а затем фильтрация начиная со строки в которой встречается 0/3:
```
R1#sh ip int
Ethernet0/0 is up, line protocol is up
  Internet address is 10.10.10.1/24
  Broadcast address is 255.255.255.255
  Address determined by configuration file
  MTU is 1500 bytes
  Helper address is not set
  Directed broadcast forwarding is disabled
  Multicast reserved groups joined: 224.0.0.10
 
/0/3     
filtering...
Ethernet0/3 is up, line protocol is up
  Internet address is 192.168.13.1/24
  Broadcast address is 255.255.255.255
  Address determined by configuration file
  MTU is 1500 bytes
```
 
Просмотр команды sh run, а затем фильтрация начиная со строки в которой встречается route-map в начале строки:
```
R1#sh run
Building configuration...
  
Current configuration : 3081 bytes
! 
! Last configuration change at 13:28:43 UTC Tue Jul 23 2013
version 15.2
service timestamps debug datetime msec
service timestamps log datetime msec
no service password-encryption
! 
hostname R1
! 
 
/^route-map
filtering...
route-map PBR_RULES permit 10
 match ip address HTTP
 set ip next-hop verify-availability 101.0.0.1 1 track 101
!
route-map PBR_RULES permit 20
 match ip address LAN2
 set ip next-hop verify-availability 102.0.0.1 1 track 102
!
```
 
## Работа с конфигурацией
 
### show run 
 
Конечно же, одна из первых вещей, которую узнает начинающий сетевик – это команда show run. Рассмотрим некоторые менее известные параметры этой команды.
 
Параметр all позволяет посмотреть значения по умолчанию, которые скрываются при просмотре show run:
```
R1# sh run all          
Building configuration...
 
Current configuration with default configurations exposed : 48318 bytes
!
! Last configuration change at 09:12:33 UTC Tue Jul 23 2013
version 15.2
parser cache
parser config partition
parser command serializer
parser maximum utilization 100
parser maximum latency 40
downward-compatible-config 15.2
no service log backtrace
no service config
no service exec-callback
no service nagle
...
 
```

Например, используя параметр all и фильтрацию вывода, можно посмотреть какие значения по умолчанию у таймаутов относящихся к трансляции адресов:
```
R1# sh run all | i ip nat
ip nat translation timeout 86400
ip nat translation tcp-timeout 86400
ip nat translation pptp-timeout 86400
ip nat translation udp-timeout 300
ip nat translation finrst-timeout 60
ip nat translation syn-timeout 60
ip nat translation dns-timeout 60
ip nat translation routemap-entry-timeout 3600
ip nat translation icmp-timeout 60
ip nat translation max-entries 0
ip nat translation max-entries all-vrf 0
ip nat translation max-entries all-host 0
ip nat translation arp-ping-timeout 60
```
 
Параметр brief позволяет скрыть некоторую информацию, которая, как правило, не нужна при просмотре. Например, открытые ключи SSH:
```
sh run brief
```
 
Параметр linenum перед каждой строкой дописывает номер строки. Может быть полезным, например, во время общения, когда надо указать о какой команде идет речь:
```
R1# sh run linenum
Building configuration...
 
Current configuration : 3109 bytes
     1 : !
     2 : ! Last configuration change at 13:02:29 UTC Tue Jul 23 2013
     3 : version 15.2
     4 : service timestamps debug datetime msec
     5 : service timestamps log datetime msec
     6 : no service password-encryption
     7 : !
     8 : hostname R1
     9 : !
```
 
Кроме того, есть несколько параметров, которые позволяют просматривать отдельные части конфигурации. Самый популярный параметр interface. Отобразить конфигурацию интерфейса fa0/1:
```
sh run interface fa0/1
 ip address 10.10.10.1 255.255.255.0
 ip nat inside
```
 
## aliases
 
Многие команды просмотра информации достаточно длинны, а использовать их нужно часто. В этом случае на помощь может прийти создание alias. Их использование сокращает время настройки, и позволяет быстрее найти ошибки, при возникновении каких-то проблем.
 
Синтаксис команды alias:
```
R1(config)# alias <режим> <название> <команда>
```
 
Например, конфигурация устройств часто достаточно большая и просматривать ее полностью неудобно. Когда вы знаете, какая часть конфигурации вас интересует, можно воспользоваться фильтрами для поиска необходимых строк. Для того чтобы фильтры удобнее было использовать, можно создать для них alias.
 
Например, фильтры применяемые к команде show run, это поиск нужных частей в конфигурации. Можно считать, что это команда find и создать, например, такие alias:
```
alias exec fb sh run | begin
alias exec fs sh run | section
alias exec fi sh run | include
alias exec fe sh run | exclude
```
 
После этого, для того чтобы, например, показать те строки конфигурации, в которых встречается ip nat, надо выполнить такую команду:
R1#fi ip nat
```
 ip nat inside
 ip nat outside
 ip nat outside
ip nat inside source route-map NAT_ISP_R2 interface Ethernet0/2 overload
ip nat inside source route-map NAT_ISP_R3 interface Ethernet0/3 overload
 action 2 cli command "clear ip nat trans *"
```
 
Для наиболее часто выполняемых команд также есть смысл создать alias. Например, для команды show ip interface brief:
```
alias exec siib sh ip int brief
```
 
Для того чтобы команду можно было выполнить в одинаковом сокращенном варианте в режиме enable и в конфигурационном, надо создавать дополнительный alias:
```
alias configure siib do sh ip int brief
```
 
Примеры alias для других популярных команд:
```
alias exec c config terminal
alias exec w write mem
alias exec sri sh run interface
alias exec cdp show cdp neighbors
alias exec sir show ip route
alias exec sr show run brief
alias exec acl show access-lists
alias configure i interface
alias interface ns no shutdown
```
 
Пример задания alias для одинаковых команд в разных режимах:
```
alias interface i ip address
alias subinterface i ip address
 
Для команды exit:
alias interface x exit
alias configure x exit
alias interface x exit
alias subinterface x exit
alias router x exit
```
 
Часто бывает так, что при выполнении команды просмотра из конфигурационных подрежимов, забывают написать "do".
 
Можно создать alias в нужных подрежимах, для того чтобы можно было писать "sh" вместо "do sh" (конечно подрежимов очень много, но создание такого alias для основных из них, может быть вполне достаточным):
```
alias configure sh do sh
alias interface sh do sh
alias subinterface sh do sh
```
 
Далее еще несколько вариантов alias.
 
Отображение команды sh ip route без шапки с расшифровкой:
```
alias exec rib show ip route | e –
 
R1#rib
Gateway of last resort is 192.168.12.2 to network 0.0.0.0
 
S*    0.0.0.0/0 [1/0] via 192.168.12.2
      10.0.0.0/8 is variably subnetted, 6 subnets, 2 masks
D        10.1.3.0/24 [90/332800] via 192.168.12.2, 2d06h, Ethernet0/2
S        10.1.3.100/32 [1/0] via 192.168.12.2
D        10.1.4.0/24 [90/332800] via 192.168.12.2, 2d06h, Ethernet0/2
S        10.1.4.100/32 [1/0] via 192.168.12.2
C        10.10.10.0/24 is directly connected, Ethernet0/0
L        10.10.10.1/32 is directly connected, Ethernet0/0
      12.0.0.0/32 is subnetted, 1 subnets
S        12.1.1.1 [1/0] via 192.168.12.2
```
 
 
Соседи CDP с IP-адресами в кратком виде:
```
alias exec cdpn sh cdp nei det | i Device|IP address|Interface
 
R1# cdpn
Device ID: sw3
  IP address: 10.10.10.2
Interface: FastEthernet0/0,  Port ID (outgoing port): FastEthernet0/0
Device ID: R2
  IP address: 192.168.12.2
Interface: FastEthernet0/2,  Port ID (outgoing port): FastEthernet0/1
Device ID: R3
  IP address: 192.168.13.3
Interface: FastEthernet0/3,  Port ID (outgoing port): FastEthernet0/1
```
 
## diff для Cisco
 
При настройке оборудования бывает, что посреди процесса настройки вас отвлекают. По возвращению вы не помните, что уже успели сделать, а если сессия завершилась и истории перед глазами нет, то сложно вспомнить какие изменения вы внесли.
 
В этом может помочь возможность сравнения файлов, которая появилась благодаря функции archive. Функция archive используется для автоматизации сохранения конфигурации. Она будет рассмотрена в одной из следующих статей. А сейчас мы воспользуемся только возможностью сравнения файлов.
 
Например, для того чтобы сравнить стартовую и текущую конфигурации, надо дать такую команду:
show archive config differences nvram:startup-config system:running-config
 
Пример выполнения команды:
```
R1# show archive config differences nvram:startup-config system:running-config
!Contextual Config Diffs:
+ip sla 14
 +icmp-echo 10.1.4.100 source-interface FastEthernet0/2
 +threshold 200
 +timeout 200
 +frequency 3
+ip sla schedule 14 life forever start-time now
+route-map PBR_RULES permit 10
 +match ip address HTTP
 +set ip next-hop verify-availability 101.0.0.1 1 track 101
+route-map PBR_RULES permit 20
 +match ip address LAN2
 +set ip next-hop verify-availability 102.0.0.1 1 track 102
-logging trap debugging
-logging 192.168.12.2
```
 
Так как команда достаточно длинная, то лучше создать для нее alias:
```
alias exec diff show archive config differences nvram:startup-config system:running-config
```
 
Аналогичным образом можно сравнивать не только текущую и стартовую конфигурации, но и другие файлы:
```
R1# show archive config differences flash:backup_200713.cfg flash:routing.cfg
!Contextual Config Diffs:
+router eigrp 1
 +network 0.0.0.0
 +network 192.168.24.0
 +redistribute ospf 1 metric 4000 3000 255 1 1500
 +redistribute static metric 30000 200 255 1 1500
+router ospf 1
 +redistribute eigrp 1 subnets
 +network 192.168.12.0 0.0.0.255 area 0
 +default-information originate
```
 
## Заключение
 
В этой статье были рассмотрены: несколько полезных команд, фильтрация вывода команд в IOS, и подсказки по поиску нужной документации.
 
В следующих статьях будут рассмотрены более продвинутые возможности IOS, в том числе функции, которые позволяют автоматизировать выполнение различных действий и создавать свои собственные сценарии.

