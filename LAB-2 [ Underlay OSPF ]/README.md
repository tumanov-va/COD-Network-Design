Домашнее задание Underlay. OSPF

Цель: Настроить OSPF для Underlay сети

Описание: Настройка протокола динамической маршрутизации OSPF для IP связанности между всеми устройствами NXOS.

Физическая схема: 

![Схема Lab1-4](https://github.com/tumanov-va/COD-Network-Design/assets/134439784/15c8eded-1473-4809-99cc-d673ce9cb9fa)

Адресное пространство для Loopback интерфейсов: 10.30.0.0/24

Адресное пространство для P2P интерфейсов: 10.30.4.0/24

Пример конфигурации (Настройки для коммутаторов Spine и Leaf идентичны в рамках данной работы):

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


Результат настройки протокола OSPF на примере коммутатора Spine-1:

    Spine-1# show ip ospf interface brief
     OSPF Process ID UNDERLAY VRF default
     Total number of interface: 4
     Interface               ID     Area            Cost   State    Neighbors Status
     Lo0                     1      0.0.0.0         1      LOOPBACK 0         up  
     Eth1/1                  4      0.0.0.0         40     P2P      1         up  
     Eth1/2                  3      0.0.0.0         40     P2P      1         up  
     Eth1/3                  2      0.0.0.0         40     P2P      1         up  
    
    
    Spine-1# show ip ospf neighbors
     OSPF Process ID UNDERLAY VRF default
     Total number of neighbors: 3
     Neighbor ID     Pri State            Up Time  Address         Interface
     Leaf-11           1 FULL/ -          00:10:10 10.30.4.1       Eth1/1 
     Leaf-12           1 FULL/ -          00:10:04 10.30.4.3       Eth1/2 
     Leaf-13           1 FULL/ -          00:09:48 10.30.4.5       Eth1/3 
    Spine-1# show ip ospf database
            OSPF Router with ID (Spine-1) (Process ID UNDERLAY VRF default)
    
                    Router Link States (Area 0.0.0.0)
    
    Link ID         ADV Router      Age        Seq#       Checksum Link Count
    10.30.0.1       Spine-1         593        0x80000007 0x7066   7   
    10.30.0.2       Spine-2         594        0x80000007 0x0aa5   7   
    10.30.0.11      Leaf-11         605        0x80000006 0xcdf8   5   
    10.30.0.12      Leaf-12         606        0x80000007 0x17a3   5   
    10.30.0.13      Leaf-13         593        0x80000006 0x644c   5   
    
    
    Spine-1# sh ip route ospf-UNDERLAY
    IP Route Table for VRF "default"
    '*' denotes best ucast next-hop
    '**' denotes best mcast next-hop
    '[x/y]' denotes [preference/metric]
    '%<string>' in via output denotes VRF <string>
    
    10.30.0.2/32, ubest/mbest: 3/0
        *via 10.30.4.1, Eth1/1, [110/81], 00:09:53, ospf-UNDERLAY, intra
        *via 10.30.4.3, Eth1/2, [110/81], 00:09:53, ospf-UNDERLAY, intra
        *via 10.30.4.5, Eth1/3, [110/81], 00:09:53, ospf-UNDERLAY, intra
    10.30.0.11/32, ubest/mbest: 1/0
        *via 10.30.4.1, Eth1/1, [110/41], 00:10:15, ospf-UNDERLAY, intra
    10.30.0.12/32, ubest/mbest: 1/0
        *via 10.30.4.3, Eth1/2, [110/41], 00:10:08, ospf-UNDERLAY, intra
    10.30.0.13/32, ubest/mbest: 1/0
        *via 10.30.4.5, Eth1/3, [110/41], 00:09:57, ospf-UNDERLAY, intra
    10.30.4.6/31, ubest/mbest: 1/0
        *via 10.30.4.1, Eth1/1, [110/80], 00:10:15, ospf-UNDERLAY, intra
    10.30.4.8/31, ubest/mbest: 1/0
        *via 10.30.4.3, Eth1/2, [110/80], 00:10:08, ospf-UNDERLAY, intra
    10.30.4.10/31, ubest/mbest: 1/0
        *via 10.30.4.5, Eth1/3, [110/80], 00:09:57, ospf-UNDERLAY, intra
