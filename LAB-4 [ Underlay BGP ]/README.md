Домашнее задание Underlay. BGP

Цель: Настроить BGP для Underlay сети

Описание: Настройка протокола динамической маршрутизации BGP [iBGP / eBGP] для IP связанности между всеми устройствами NXOS.

Физическая схема: 

![Схема Lab1-4](https://github.com/tumanov-va/COD-Network-Design/assets/134439784/15c8eded-1473-4809-99cc-d673ce9cb9fa)

  Адресное пространство для Loopback интерфейсов: 10.30.0.0/24
  Адресное пространство для P2P интерфейсов: 10.30.4.0/24

Приложения к лабораторной работе:
  
        LAB-4 / [Underlay eBGP Config].txt - Настройка устройств
        LAB-4 / [Underlay eBGP Output].txt - Результат настройки протокола eBGP на устройствах NXOS
        LAB-4 / [Underlay iBGP Config].txt - Настройка устройств
        LAB-4 / [Underlay iBGP Output].txt - Результат настройки протокола iBGP на устройствах NXOS
        LAB-4 / Схема Lab1-4.png - Физическая схема
 
                                
Пример конфигурации для коммутаторов Spine и Leaf):

                                         ######          EBGP         #######

    Spine ASn = 65030
    Leaf11 ASn = 65031
    Leaf12 ASn = 65031
    Leaf13 ASn = 65033

###########################################################################################################

      hostname Spine-1
      
      feature bgp
      feature bfd
      
      ip host Spine-1 10.30.0.1
      ip host Spine-2 10.30.0.2
      ip host Leaf-11 10.30.0.11
      ip host Leaf-12 10.30.0.12
      ip host Leaf-13 10.30.0.13
            
      ip as-path access-list AS-PATH-LIST seq 10 permit "65031:65999"
      route-map AS_FILTER permit 10
        match as-path AS-PATH-LIST 
      route-map REDISTRIBUTE_CONNECTED permit 10
        match interface loopback0 
      key chain UNDERLAY_KeyChain
        key 1
          key-string 7 073f007f7d3e363733
          cryptographic-algorithm HMAC-SHA-256
      
      interface Ethernet1/1
        description # Leaf-11 # 10.30.0.11 # Port # Ethernet1/1
        no switchport
        mtu 9216
        no ip redirects
        ip address 10.30.4.0/31
        no shutdown
      
      interface Ethernet1/2
        description # Leaf-12 # 10.30.0.12 # Port # Ethernet1/1
        no switchport
        mtu 9216
        no ip redirects
        ip address 10.30.4.2/31
        no shutdown
      
      interface Ethernet1/3
        description # Leaf-13 # 10.30.0.13 # Port # Ethernet1/1
        no switchport
        mtu 9216
        no ip redirects
        ip address 10.30.4.4/31
        no shutdown
      
      interface loopback0
        description Router_ID
        ip address 10.30.0.1/32
      
      boot nxos bootflash:/nxos.9.3.8.bin sup-1
      router bgp 65030
        router-id 10.30.0.1
        bestpath as-path multipath-relax
        maxas-limit 3
        log-neighbor-changes
        address-family ipv4 unicast
          redistribute direct route-map REDISTRIBUTE_CONNECTED
        neighbor 10.30.4.0/24 remote-as route-map AS_FILTER
          bfd
          description LEAF
          password 3 5c0c2e6087f1e976cc0f58d4a90b33e6
          timers 1 3
          address-family ipv4 unicast

###########################################################################################################
 
    hostname Leaf-11
    
    feature bgp
    feature bfd
    
    ip host Spine-1 10.30.0.1
    ip host Spine-2 10.30.0.2
    ip host Leaf-11 10.30.0.11
    ip host Leaf-12 10.30.0.12
    ip host Leaf-13 10.30.0.13
        
    route-map REDISTRIBUTE_CONNECTED permit 10
      match interface loopback0 
    key chain UNDERLAY_KeyChain
      key 1
        key-string 7 073f007f7d3e363733
        cryptographic-algorithm HMAC-SHA-256
    
    interface Ethernet1/1
      description # Spine-1 # 10.30.0.1 # Port # Ethernet1/1
      no switchport
      mtu 9216
      no ip redirects
      ip address 10.30.4.1/31
      no shutdown
    
    interface Ethernet1/2
      description # Spine-1 # 10.30.0.2 # Port # Ethernet1/2
      no switchport
      mtu 9216
      no ip redirects
      ip address 10.30.4.7/31
      no shutdown
    
    interface loopback0
      description Router_ID
      ip address 10.30.0.11/32
    icam monitor scale
    
    router bgp 65031
      router-id 10.30.0.11
      log-neighbor-changes
      address-family ipv4 unicast
        redistribute direct route-map REDISTRIBUTE_CONNECTED
      template peer SPINE
        bfd
        remote-as 65030
        description LEAF
        password 3 5c0c2e6087f1e976cc0f58d4a90b33e6
        timers 1 3
        address-family ipv4 unicast
      neighbor 10.30.4.0
        inherit peer SPINE
        description SPINE-1
      neighbor 10.30.4.6
        inherit peer SPINE
        description SPINE-2

Результат настройки протокола BGP (eBGP) на примере коммутаторов Spine-1 и Leaf-12:
                                         
      Spine-1# show ip bgp summary
      BGP summary information for VRF default, address family IPv4 Unicast
      BGP router identifier 10.30.0.1, local AS number 65030
      BGP table version is 7, IPv4 Unicast config peers 3, capable peers 2
      3 network entries and 3 paths using 732 bytes of memory
      BGP attribute entries [3/516], BGP AS path entries [2/12]
      BGP community entries [0/0], BGP clusterlist entries [0/0]
      
      Neighbor        V    AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
      10.30.4.1       4 65031     421     420        7    0    0 00:06:59 1         
      10.30.4.3       4 65032     427     431        7    0    0 00:06:54 1         
      
      Spine-1# show ip bgp
      BGP routing table information for VRF default, address family IPv4 Unicast
      BGP table version is 7, Local Router ID is 10.30.0.1
      Status: s-suppressed, x-deleted, S-stale, d-dampened, h-history, *-valid, >-best
      Path type: i-internal, e-external, c-confed, l-local, a-aggregate, r-redist, I-i
      njected
      Origin codes: i - IGP, e - EGP, ? - incomplete, | - multipath, & - backup, 2 - b
      est2
      
         Network            Next Hop            Metric     LocPrf     Weight Path
      *>r10.30.0.1/32       0.0.0.0                  0        100      32768 ?
      *>e10.30.0.11/32      10.30.4.1                0                     0 65031 ?
      *>e10.30.0.12/32      10.30.4.3                0                     0 65032 ?
      
      Spine-1# show ip route bgp
      IP Route Table for VRF "default"
      '*' denotes best ucast next-hop
      '**' denotes best mcast next-hop
      '[x/y]' denotes [preference/metric]
      '%<string>' in via output denotes VRF <string>
      
      10.30.0.11/32, ubest/mbest: 1/0
          *via 10.30.4.1, [20/0], 00:07:17, bgp-65030, external, tag 65031
      10.30.0.12/32, ubest/mbest: 1/0
          *via 10.30.4.3, [20/0], 00:07:29, bgp-65030, external, tag 65032

#################################
#################################

      Leaf-12# show ip bgp summary
      show ip bgp
      show ip route bgpBGP summary information for VRF default, address family IPv4 Unicast
      BGP router identifier 10.30.0.12, local AS number 65032
      BGP table version is 75, IPv4 Unicast config peers 2, capable peers 2
      5 network entries and 6 paths using 1340 bytes of memory
      BGP attribute entries [4/688], BGP AS path entries [3/26]
      BGP community entries [0/0], BGP clusterlist entries [0/0]
      
      Neighbor        V    AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
      10.30.4.2       4 65030    2057    2058       75    0    0 00:22:08 2         
      10.30.4.8       4 65030    6148    6144       75    0    0 00:22:07 3         
      Leaf-12# show ip bgp
      BGP routing table information for VRF default, address family IPv4 Unicast
      BGP table version is 75, Local Router ID is 10.30.0.12
      Status: s-suppressed, x-deleted, S-stale, d-dampened, h-history, *-valid, >-best
      Path type: i-internal, e-external, c-confed, l-local, a-aggregate, r-redist, I-i
      njected
      Origin codes: i - IGP, e - EGP, ? - incomplete, | - multipath, & - backup, 2 - b
      est2
      
         Network            Next Hop            Metric     LocPrf     Weight Path
      *>e10.30.0.1/32       10.30.4.2                0                     0 65030 ?
      *>e10.30.0.2/32       10.30.4.8                0                     0 65030 ?
      *>e10.30.0.11/32      10.30.4.2                                      0 65030 65031 ?
      * e                   10.30.4.8                                      0 65030 65031 ?
      *>r10.30.0.12/32      0.0.0.0                  0        100      32768 ?
      *>e10.30.0.13/32      10.30.4.8                                      0 65030 65033 ?
      
      Leaf-12# show ip route bgp
      IP Route Table for VRF "default"
      '*' denotes best ucast next-hop
      '**' denotes best mcast next-hop
      '[x/y]' denotes [preference/metric]
      '%<string>' in via output denotes VRF <string>
      
      10.30.0.1/32, ubest/mbest: 1/0
          *via 10.30.4.2, [20/0], 00:22:09, bgp-65032, external, tag 65030
      10.30.0.2/32, ubest/mbest: 1/0
          *via 10.30.4.8, [20/0], 00:22:08, bgp-65032, external, tag 65030
      10.30.0.11/32, ubest/mbest: 1/0
          *via 10.30.4.2, [20/0], 00:22:09, bgp-65032, external, tag 65030
      10.30.0.13/32, ubest/mbest: 1/0
          *via 10.30.4.8, [20/0], 00:22:08, bgp-65032, external, tag 65030	
