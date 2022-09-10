# Маршрутизация между VLAN

### Задание

1. [Выполнить начальную настройку устройств](README.md#%D0%BD%D0%B0%D1%87%D0%B0%D0%BB%D1%8C%D0%BD%D0%B0%D1%8F-%D0%BD%D0%B0%D1%81%D1%82%D1%80%D0%BE%D0%B9%D0%BA%D0%B0-%D1%83%D1%81%D1%82%D1%80%D0%BE%D0%B9%D1%81%D1%82%D0%B2%D0%B0)
2. [Настроить VLAN на коммутаторах](README.md#%D1%81%D0%BE%D0%B7%D0%B4%D0%B0%D0%BD%D0%B8%D0%B5-vlan-%D0%B8-%D0%BD%D0%B0%D0%B7%D0%BD%D0%B0%D1%87%D0%B5%D0%BD%D0%B8%D0%B5-%D0%BF%D0%BE%D1%80%D1%82%D0%BE%D0%B2-%D0%BD%D0%B0-%D0%BA%D0%BE%D0%BC%D0%BC%D1%83%D1%82%D0%B0%D1%82%D0%BE%D1%80%D0%B0%D1%85)
3. [Организовать маршрутизацию в сети](https://github.com/v925659/network_engineer_otus/edit/main/lab01/README.md#%D0%BD%D0%B0%D1%81%D1%82%D1%80%D0%BE%D0%B9%D0%BA%D0%B0-vlan-%D0%BD%D0%B0-%D0%BC%D0%B0%D1%80%D1%88%D1%80%D1%83%D1%82%D0%B8%D0%B7%D0%B0%D1%82%D0%BE%D1%80%D0%B5)

### Исходные данные

#### Схема сети L1/L2
![](network_lab01.svg)
#### Таблица IP адресации
Устройство | Интерфейс  | IP Адрес  | Маска подсети | Шлюз
---------- | ---------- | --------- | ------------- | ----
R1 | G0/0/1.3 | 192.168.3.1 | 255.255.255.0 | N/A
R1 | G0/0/1.4 | 192.168.4.1 | 255.255.255.0 | N/A
R1 | G0/0/1.8 | N/A | N/A | N/A
S1 | VLAN 3 | 192.168.3.11 | 255.255.255.0 | 192.168.3.1
S2 | VLAN 3 | 192.168.3.12 | 255.255.255.0 | 192.168.3.1
PC-A | NIC | 192.168.3.3 | 255.255.255.0 | 192.168.3.1
PC-B | NIC | 192.168.4.3 | 255.255.255.0 | 192.168.4.1
#### Таблица VLAN
VLAN | Имя | Интерфейсы
---- | --- | ----------
3	| Management | S1: VLAN 3 <br/> S2: VLAN 3 <br/> S1: F0/6
4	| Operations |	S2: F0/18
7 | ParkingLot |	S1: F0/2-4, F0/7-24, G0/1-2 <br/> S2: F0/2-17, F0/19-24, G0/1-2 
8 |	Native |	N/A

### Начальная настройка устройства

Переход в привилегированный режим (privileged mode)
```
> enable
```
Переход в режим глобальной конфигурации (global configuration mode)
```
# configure terminal
```
Смена имени устройства
```
(config)# hostname <string>
```
Отключение поиска по DNS
```
(config)# no ip domain-lookup
```
Установка пароля на доступ режим глобальной конфигурации (global configuration mode)
```
(config)# enable password <password>
```
Установка пароля на вход через консольный провод
```
(config)# line console 0
(config-line)# password <password>
(config-line)# login
```
Установка пароля на вход через Telnet или SSH
```
(config)# line vty 0 4
(config-line)# password <password>
(config-line)# login
(config-line)# exit
```
Включение службы шифрования паролей
```
(config)# service password-encryption
```
Создание баннера при подключении к устройству
```
(config)# banner motd # text #
```
Установка часов на устройстве
```
(config)# clock set 20:22:00 1 sep 2022
```
Просмотр текущей конфигурации
```
# show running-config
```
Копирование текущей конфигурации в стартовую
```
# copy running-config startup-config
```
### Создание VLAN и назначение портов на коммутаторах

Создание VLAN с номером 3 и именем «Management»
```
(config)# vlan 3
(config-vlan)# name Management
```
Настройка интерфейса управления 
```
(config)# interface FastEthernet0/0
(config-if)# no shutdown
(config-if)# ip address 192.168.3.11 255.255.255.0
(config-if)# switchport access vlan 2
(config-if)# exit
(config)# ip default-gateway 192.168.3.1
```
Настройка access порта
```
(config)# interface FastEthernet0/6
(config-if)# no shutdown
(config-if)# switchport mode access
(config-if)# switchport access vlan 3
(config-if)# exit
```
Выбор диапазона access портов для настройки VLAN и принудительного отключения
```
(config)# interface range FastEthernet 0/2 - 4
(config-if-range)# switchport mode access
(config-if-range)# switchport access vlan 7
(config-if-range)# shutdown
(config-if-range)# exit
```
Просмотр информации обо всех VLAN
```
# show vlan brief
```
Настройка trunk порта (статический транк)
```
(config)# interface FastEthernet0/1
(config-if)# no shutdown
(config-if)# switchport mode trunk
(config-if)# switchport trunk allowed vlan 3-4,8
(config-if)# switchport trunk native vlan 8
(config-if)# exit
```
Просмотр информации обо всех trunk портах
```
# show interfaces trunk
```
Конфигурационные файлы коммутаторов [S1](../blob/main/lab01/configs/S1) и [S2](../blob/main/lab01/configs/S2) и пример конфигурации основных портов: <br/>
S1
```
!
interface FastEthernet0/1
 switchport trunk native vlan 8
 switchport trunk allowed vlan 3-4,8
 switchport mode trunk
!
interface FastEthernet0/5
 switchport trunk native vlan 8
 switchport trunk allowed vlan 3-4,8
 switchport mode trunk
!
interface FastEthernet0/6
 switchport access vlan 3
 switchport mode access
!
```
S2
```
!
interface FastEthernet0/1
 switchport trunk native vlan 8
 switchport trunk allowed vlan 3-4,8
 switchport mode trunk
!
interface FastEthernet0/18
 switchport access vlan 4
 switchport mode access
!
```

### Настройка VLAN на маршрутизаторе

Настройка логического подынтерфейса для VLAN 3
```
(config)# interface fa0/0.3
(config-subif)# encapsulation dot1q 3
(config-subif)# description Managment
(config-subif)# ip address 192.168.3.1 255.255.255.0
(config-subif)# exit
```
Настройка логического подынтерфейс для передачи не тегированного трафика (native VLAN 8)
```
(config-subif)# encapsulation dot1q 8 native
```
Просмотр информации обо все интерфейсах маршрутизатора 
```
# show ip interface brief
```
Конфигурационный файл маршрутизатора [R1](../blob/main/lab01/configs/R1) и пример конфигураци порта G0/0/1 и его подынтерфейсов:
```
!
interface GigabitEthernet0/0/1
 no ip address
 duplex auto
 speed auto
!
interface GigabitEthernet0/0/1.3
 description Managment
 encapsulation dot1Q 3
 ip address 192.168.3.1 255.255.255.0
!
interface GigabitEthernet0/0/1.4
 description Operation
 encapsulation dot1Q 4
 ip address 192.168.4.1 255.255.255.0
!
interface GigabitEthernet0/0/1.8
 description Native
 encapsulation dot1Q 8 native
 no ip address
!
```
### Проверка работоспособности сети
Все команды выполняются на PC-A
```
C:\>ipconfig

FastEthernet0 Connection:(default port)

   Connection-specific DNS Suffix..: 
   Link-local IPv6 Address.........: FE80::20B:BEFF:FEE0:C84C
   IPv6 Address....................: ::
   IPv4 Address....................: 192.168.3.3
   Subnet Mask.....................: 255.255.255.0
   Default Gateway.................: ::
                                     192.168.3.1

Bluetooth Connection:

   Connection-specific DNS Suffix..: 
   Link-local IPv6 Address.........: ::
   IPv6 Address....................: ::
   IPv4 Address....................: 0.0.0.0
   Subnet Mask.....................: 0.0.0.0
   Default Gateway.................: ::
                                     0.0.0.0

C:\>ping 192.168.3.1

Pinging 192.168.3.1 with 32 bytes of data:

Reply from 192.168.3.1: bytes=32 time<1ms TTL=255
Reply from 192.168.3.1: bytes=32 time<1ms TTL=255
Reply from 192.168.3.1: bytes=32 time=7ms TTL=255
Reply from 192.168.3.1: bytes=32 time<1ms TTL=255

Ping statistics for 192.168.3.1:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
Approximate round trip times in milli-seconds:
    Minimum = 0ms, Maximum = 7ms, Average = 1ms

C:\>ping 192.168.4.3

Pinging 192.168.4.3 with 32 bytes of data:

Reply from 192.168.4.3: bytes=32 time=7ms TTL=127
Reply from 192.168.4.3: bytes=32 time<1ms TTL=127
Reply from 192.168.4.3: bytes=32 time<1ms TTL=127
Reply from 192.168.4.3: bytes=32 time=10ms TTL=127

Ping statistics for 192.168.4.3:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
Approximate round trip times in milli-seconds:
    Minimum = 0ms, Maximum = 10ms, Average = 4ms

C:\>ping 192.168.3.12

Pinging 192.168.3.12 with 32 bytes of data:

Reply from 192.168.3.12: bytes=32 time<1ms TTL=255
Reply from 192.168.3.12: bytes=32 time<1ms TTL=255
Reply from 192.168.3.12: bytes=32 time<1ms TTL=255
Reply from 192.168.3.12: bytes=32 time<1ms TTL=255

Ping statistics for 192.168.3.12:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
Approximate round trip times in milli-seconds:
    Minimum = 0ms, Maximum = 0ms, Average = 0ms

```

