Домашнее задание Underlay. IS-IS

Цель: Настроить IS-IS для Underlay сети

Описание: Настройка протокола динамической маршрутизации IS-IS для IP связанности между всеми устройствами NXOS.

Физическая схема: 

![Схема Lab1-4](https://github.com/tumanov-va/COD-Network-Design/assets/134439784/15c8eded-1473-4809-99cc-d673ce9cb9fa)

Адресное пространство для Loopback интерфейсов: 10.30.0.0/24

Адресное пространство для P2P интерфейсов: 10.30.4.0/24

Пример конфигурации (Настройки для коммутаторов Spine и Leaf идентичны в рамках данной работы):

        hostname Leaf-12
        
        feature isis
        feature lldp
        feature bfd
        
        ip host Spine-1 10.30.0.1
        ip host Spine-2 10.30.0.2
        ip host Leaf-11 10.30.0.11
        ip host Leaf-12 10.30.0.12
        ip host Leaf-13 10.30.0.13
        
        key chain ISIS_KeyChain
          key 1
            key-string 7 073f007f7d3e363733
            cryptographic-algorithm HMAC-SHA-256
        vrf context management
        
        interface Ethernet1/1
          description # Spine-1 # 10.30.0.1 # Port # Ethernet1/1
          no switchport
          mtu 9216
          no ip redirects
          ip address 10.30.4.3/31
          no isis hello-padding always
          isis network point-to-point
          isis circuit-type level-2
          isis authentication-type md5 level-2
          isis authentication key-chain ISIS_KeyChain level-2
          ip router isis UNDERLAY
          no shutdown
        
        interface Ethernet1/2
          description # Spine-1 # 10.30.0.2 # Port # Ethernet1/2
          no switchport
          mtu 9216
          no ip redirects
          ip address 10.30.4.9/31
          no isis hello-padding always
          isis network point-to-point
          isis circuit-type level-2
          isis authentication-type md5 level-2
          isis authentication key-chain ISIS_KeyChain level-2
          ip router isis UNDERLAY
          no shutdown
        
        interface loopback0
          description Router_ID
          ip address 10.30.0.12/32
          ip router isis UNDERLAY
        
        router isis UNDERLAY
          net 49.0002.0100.3000.0012.00
          is-type level-2
          log-adjacency-changes
          authentication-type md5 level-2
          authentication key-chain ISIS_KeyChain level-2
          address-family ipv4 unicast

Результат настройки протокола IS-IS на примере коммутаторов Spine-1 и Leaf-12:

    SPINE-1# show isis interface brief
    IS-IS process: UNDERLAY VRF: default
    Interface    Type  Idx State        Circuit   MTU  Metric  Priority  Adjs/AdjsUp
                                                       L1  L2  L1  L2    L1    L2  
    --------------------------------------------------------------------------------
    Topology: TopoID: 0
    loopback0    Loop  1     Up/Ready   0x01/L2   1500 1   1   64  64    0/0   0/0 
    Topology: TopoID: 0
    Ethernet1/1  P2P   4     Up/Ready   0x01/L2   9216 40  40  64  64    0/0   1/1 
    Topology: TopoID: 0
    Ethernet1/2  P2P   3     Up/Ready   0x01/L2   9216 40  40  64  64    0/0   1/1 
    Topology: TopoID: 0
    Ethernet1/3  P2P   2     Up/Ready   0x01/L2   9216 40  40  64  64    0/0   1/1 
    
    SPINE-1# show isis adjacency
    IS-IS process: UNDERLAY VRF: default
    IS-IS adjacency database:
    Legend: '!': No AF level connectivity in given topology
    System ID       SNPA            Level  State  Hold Time  Interface
    Leaf-11         N/A             2      UP     00:00:24   Ethernet1/1
    Leaf-12         N/A             2      UP     00:00:24   Ethernet1/2
    Leaf-13         N/A             2      UP     00:00:28   Ethernet1/3
    
    SPINE-1# show isis database
    IS-IS Process: UNDERLAY LSP database VRF: default
    IS-IS Level-1 Link State Database
      LSPID                 Seq Number   Checksum  Lifetime   A/P/O/T
    
    IS-IS Level-2 Link State Database
      LSPID                 Seq Number   Checksum  Lifetime   A/P/O/T
      SPINE-1.00-00       * 0x00000034   0x49D9    1152       0/0/0/3
      SPINE-2.00-00         0x00000026   0x2871    1029       0/0/0/3
      Leaf-11.00-00         0x00000026   0x57DD    1146       0/0/0/3
      Leaf-12.00-00         0x00000026   0xCFE6    1180       0/0/0/3
      Leaf-13.00-00         0x00000027   0xC627    1118       0/0/0/3
    
    SPINE-1# sh ip route isis-UNDERLAY
    IP Route Table for VRF "default"
    '*' denotes best ucast next-hop
    '**' denotes best mcast next-hop
    '[x/y]' denotes [preference/metric]
    '%<string>' in via output denotes VRF <string>
    
    10.30.0.2/32, ubest/mbest: 3/0
        *via 10.30.4.1, Eth1/1, [115/81], 00:37:37, isis-UNDERLAY, L2
        *via 10.30.4.3, Eth1/2, [115/81], 00:37:31, isis-UNDERLAY, L2
        *via 10.30.4.5, Eth1/3, [115/81], 00:37:37, isis-UNDERLAY, L2
    10.30.0.11/32, ubest/mbest: 1/0
        *via 10.30.4.1, Eth1/1, [115/41], 00:37:37, isis-UNDERLAY, L2
    10.30.0.12/32, ubest/mbest: 1/0
        *via 10.30.4.3, Eth1/2, [115/41], 00:37:31, isis-UNDERLAY, L2
    10.30.0.13/32, ubest/mbest: 1/0
        *via 10.30.4.5, Eth1/3, [115/41], 00:37:37, isis-UNDERLAY, L2
    10.30.4.6/31, ubest/mbest: 1/0
        *via 10.30.4.1, Eth1/1, [115/80], 00:37:37, isis-UNDERLAY, L2
    10.30.4.8/31, ubest/mbest: 1/0
        *via 10.30.4.3, Eth1/2, [115/80], 00:37:31, isis-UNDERLAY, L2
    10.30.4.10/31, ubest/mbest: 1/0
        *via 10.30.4.5, Eth1/3, [115/80], 00:37:37, isis-UNDERLAY, L2


    Leaf-12# show isis interface brief
    show isis database
    IS-IS process: UNDERLAY VRF: default
    Interface    Type  Idx State        Circuit   MTU  Metric  Priority  Adjs/AdjsUp
                                                       L1  L2  L1  L2    L1    L2  
    --------------------------------------------------------------------------------
    Topology: TopoID: 0
    loopback0    Loop  1     Up/Ready   0x01/L2   1500 1   1   64  64    0/0   0/0 
    Topology: TopoID: 0
    Ethernet1/1  P2P   3     Up/Ready   0x01/L2   9216 40  40  64  64    0/0   1/1 
    Topology: TopoID: 0
    Ethernet1/2  P2P   2     Up/Ready   0x01/L2   9216 40  40  64  64    0/0   1/1 
    
    Leaf-12# show isis adjacency
    IS-IS process: UNDERLAY VRF: default
    IS-IS adjacency database:
    Legend: '!': No AF level connectivity in given topology
    System ID       SNPA            Level  State  Hold Time  Interface
    SPINE-1         N/A             2      UP     00:00:31   Ethernet1/1
    SPINE-2         N/A             2      UP     00:00:31   Ethernet1/2
    
    Leaf-12# show isis database
    IS-IS Process: UNDERLAY LSP database VRF: default
    IS-IS Level-1 Link State Database
      LSPID                 Seq Number   Checksum  Lifetime   A/P/O/T
    
    IS-IS Level-2 Link State Database
      LSPID                 Seq Number   Checksum  Lifetime   A/P/O/T
      SPINE-1.00-00         0x00000034   0x49D9    835        0/0/0/3
      SPINE-2.00-00         0x00000026   0x2871    713        0/0/0/3
      Leaf-11.00-00         0x00000026   0x57DD    829        0/0/0/3
      Leaf-12.00-00       * 0x00000026   0xCFE6    864        0/0/0/3
      Leaf-13.00-00         0x00000027   0xC627    800        0/0/0/3
    
    Leaf-12# sh ip route isis-UNDERLAY
    IP Route Table for VRF "default"
    '*' denotes best ucast next-hop
    '**' denotes best mcast next-hop
    '[x/y]' denotes [preference/metric]
    '%<string>' in via output denotes VRF <string>
    
    10.30.0.1/32, ubest/mbest: 1/0
        *via 10.30.4.2, Eth1/1, [115/41], 00:42:40, isis-UNDERLAY, L2
    10.30.0.2/32, ubest/mbest: 1/0
        *via 10.30.4.8, Eth1/2, [115/41], 01:02:51, isis-UNDERLAY, L2
    10.30.0.11/32, ubest/mbest: 2/0
        *via 10.30.4.2, Eth1/1, [115/81], 00:42:40, isis-UNDERLAY, L2
        *via 10.30.4.8, Eth1/2, [115/81], 01:02:51, isis-UNDERLAY, L2
    10.30.0.13/32, ubest/mbest: 2/0
        *via 10.30.4.2, Eth1/1, [115/81], 00:42:40, isis-UNDERLAY, L2
        *via 10.30.4.8, Eth1/2, [115/81], 01:02:51, isis-UNDERLAY, L2
    10.30.4.0/31, ubest/mbest: 1/0
        *via 10.30.4.2, Eth1/1, [115/80], 00:42:40, isis-UNDERLAY, L2
    10.30.4.4/31, ubest/mbest: 1/0
        *via 10.30.4.2, Eth1/1, [115/80], 00:42:40, isis-UNDERLAY, L2
    10.30.4.6/31, ubest/mbest: 1/0
        *via 10.30.4.8, Eth1/2, [115/80], 01:02:51, isis-UNDERLAY, L2
    10.30.4.10/31, ubest/mbest: 1/0
        *via 10.30.4.8, Eth1/2, [115/80], 01:02:51, isis-UNDERLAY, L2




