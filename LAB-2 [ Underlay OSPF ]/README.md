Домашнее задание Underlay. OSPF

Цель: Настроить OSPF для Underlay сети

Описание: Настройка протокола динамической маршрутизации OSPF для IP связанности между всеми устройствами NXOS.

Физическая схема: 

![Схема Lab1-4](https://github.com/tumanov-va/COD-Network-Design/assets/134439784/15c8eded-1473-4809-99cc-d673ce9cb9fa)

Адресное пространство для Loopback интерфейсов: 10.30.0.0/24

Адресное пространство для P2P интерфейсов: 10.30.4.0/24

Пример конфигурации (Настройки для коммутаторов Spine и Leaf идентичны в рамках данной работы)

hostname Spine-1
!
feature ospf
feature lldp
feature bfd
!
ip host Spine-2 10.30.0.2
ip host Leaf-11 10.30.0.11
ip host Leaf-12 10.30.0.12
ip host Leaf-13 10.30.0.13
ip host Spine-1 10.30.0.1
!
key chain OSPF_KeyChain
  key 1
    key-string 7 073f007f7d3e363733
    cryptographic-algorithm HMAC-SHA-256
vrf context management
!
interface Ethernet1/1
  description # Leaf-11 # 10.30.0.11 # Port # Ethernet1/1
  no switchport
  mtu 9216
  no ip redirects
  ip address 10.30.4.0/31
  ip ospf authentication message-digest
  ip ospf authentication key-chain OSPF_KeyChain
  ip ospf network point-to-point
  no ip ospf passive-interface
  ip router ospf UNDERLAY area 0.0.0.0
  ip ospf bfd
  no shutdown
!
interface Ethernet1/2
  description # Leaf-12 # 10.30.0.12 # Port # Ethernet1/1
  no switchport
  mtu 9216
  no ip redirects
  ip address 10.30.4.2/31
  ip ospf authentication message-digest
  ip ospf authentication key-chain OSPF_KeyChain
  ip ospf network point-to-point
  no ip ospf passive-interface
  ip router ospf UNDERLAY area 0.0.0.0
  ip ospf bfd
  no shutdown
!
interface Ethernet1/3
  description # Leaf-13 # 10.30.0.13 # Port # Ethernet1/1
  no switchport
  mtu 9216
  no ip redirects
  ip address 10.30.4.4/31
  ip ospf authentication message-digest
  ip ospf authentication key-chain OSPF_KeyChain
  ip ospf network point-to-point
  no ip ospf passive-interface
  ip router ospf UNDERLAY area 0.0.0.0
  ip ospf bfd
  no shutdown
!
interface loopback0
  description Router_ID
  ip address 10.30.0.1/32
  ip router ospf UNDERLAY area 0.0.0.0
!
router ospf UNDERLAY
  bfd
  router-id 10.30.0.1
  area 0.0.0.0 authentication message-digest
  name-lookup
!


Приложения к лабораторной работе:

LAB-2 / [Underlay OSPF Config].txt - Настройка устройств

LAB-2 / [Underlay OSPF Output].txt - Результат настройки протокола OSPF на устройствах NXOS


