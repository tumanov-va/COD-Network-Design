Цель:  Настроить Overlay на основе VxLAN EVPN для L2 связанности между клиентами

Описание: Настроить BGP peering между Leaf и Spine в AF l2vpn evpn

Настроена связанность между клиентами в первой зоне

Адресное пространство для построения Underlay 

      Loopback интерфейсы: 10.30.0.0/24 [ Lo0 ]
      P2P интерфейсы SPINE-LEAF: 10.30.4.0/24

Адресное пространство для построения Overlay:

      Loopback интерфейсы: 10.30.1.0/24 [ Lo1 ]
      
      Client1_Net: 10.30.10.0/24
      Client1_Vlan_Id: 10 (VPC-1,VPC-3)
      
      Client2_Net: 10.30.20.0/24
      Client2_Vlan_Id: 20 (VPC-2,VPC-4)

Физическая схема
![Схема](https://github.com/tumanov-va/COD-Network-Design/assets/134439784/5b6056df-74e4-4162-b7ab-7685eea96fa5)

Приложения к лабораторной работе:

     LAB-5 / [ VxLAN EVPN L2 Config].txt - Настройка устройств
     LAB-5 / [ VxLAN EVPN L2 Output].txt - Результат настройки VxLAN EVPN L2

Пример конфигурации для коммутаторов Spine и Leaf:

      Spine's ASn = 65030
      Leaf11 ASn = 65031
      Leaf12 ASn = 65031
      Leaf13 ASn = 65033

###########################################################################################################

      hostname Spine-1
      
      nv overlay evpn
      feature bgp
      feature fabric forwarding
      feature vn-segment-vlan-based
      feature bfd
      feature nv overlay
      
      ip host Spine-1 10.30.0.1
      ip host Spine-2 10.30.0.2
      ip host Leaf-11 10.30.0.11
      ip host Leaf-12 10.30.0.12
      ip host Leaf-13 10.30.0.13
      
      ip as-path access-list AS-PATH-LIST seq 10 permit "65031:65999"
      route-map AS_FILTER permit 10
        match as-path AS-PATH-LIST 
      route-map REDISTRIBUTE_CONNECTED permit 10
        match interface loopback0 loopback1 
      route-map next-hop-unchanged permit 10
        set ip next-hop unchanged
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
        description Router_ID # ControlPlane_EVPN
        ip address 10.30.0.1/32
      
      interface loopback1
        description Router_ID # DataPlane_VxLAN
        ip address 10.30.1.1/32
      icam monitor scale
      
      router bgp 65030
        router-id 10.30.0.1
        bestpath as-path multipath-relax
        maxas-limit 3
        log-neighbor-changes
        address-family ipv4 unicast
          redistribute direct route-map REDISTRIBUTE_CONNECTED
        address-family l2vpn evpn
          retain route-target all
        neighbor 10.30.1.0/24 remote-as route-map AS_FILTER
          description LEAF_OVERLAY
          password 3 5c0c2e6087f1e976cc0f58d4a90b33e6
          update-source loopback1
          ebgp-multihop 2
          address-family l2vpn evpn
            send-community
            send-community extended
            route-map next-hop-unchanged out
        neighbor 10.30.4.0/24 remote-as route-map AS_FILTER
          description LEAF
          password 3 5c0c2e6087f1e976cc0f58d4a90b33e6
          timers 3 15
          address-family ipv4 unicast
      evpn

#####################
#####################

      hostname Leaf-13
      
      nv overlay evpn
      feature bgp
      feature fabric forwarding
      feature vn-segment-vlan-based
      feature bfd
      feature nv overlay
      
      ip host Spine-1 10.30.0.1
      ip host Spine-2 10.30.0.2
      ip host Leaf-11 10.30.0.11
      ip host Leaf-12 10.30.0.12
      ip host Leaf-13 10.30.0.13
      
      vlan 1,10,20
      vlan 10
        vn-segment 10000
      vlan 20
        vn-segment 20000
      
      route-map REDISTRIBUTE_CONNECTED permit 10
        match interface loopback0 loopback1 
      key chain UNDERLAY_KeyChain
        key 1
          key-string 7 073f007f7d3e363733
          cryptographic-algorithm HMAC-SHA-256
      
      hardware access-list tcam region racl 512
      hardware access-list tcam region arp-ether 256 double-wide
      
      interface nve1
        no shutdown
        host-reachability protocol bgp
        source-interface loopback1
        member vni 10000
          suppress-arp
          ingress-replication protocol bgp
        member vni 20000
          suppress-arp
          ingress-replication protocol bgp
      
      interface Ethernet1/1
        description # Spine-1 # 10.30.0.1 # Port # Ethernet1/1
        no switchport
        mtu 9216
        no ip redirects
        ip address 10.30.4.5/31
        no shutdown
      
      interface Ethernet1/2
        description # Spine-1 # 10.30.0.2 # Port # Ethernet1/2
        no switchport
        mtu 9216
        no ip redirects
        ip address 10.30.4.11/31
        no shutdown
      
      interface Ethernet1/6
        description # Client-3 # 10.30.10.11 #
        switchport access vlan 10
      
      interface Ethernet1/7
        description # Client-4 # 10.30.20.11 #
        switchport access vlan 20
      
      
      interface loopback0
        description Router_ID # ControlPlane_EVPN
        ip address 10.30.0.13/32
      
      interface loopback1
        description Router_ID # DataPlane_VxLAN
        ip address 10.30.1.13/32
      
      boot nxos bootflash:/nxos.9.3.10.bin sup-1
      router bgp 65033
        router-id 10.30.0.13
        bestpath as-path multipath-relax
        log-neighbor-changes
        address-family ipv4 unicast
          redistribute direct route-map REDISTRIBUTE_CONNECTED
        template peer SPINE
          remote-as 65030
          description LEAF
          password 3 5c0c2e6087f1e976cc0f58d4a90b33e6
          timers 3 15
          address-family ipv4 unicast
        template peer SPINE_OVERLAY
          remote-as 65030
          description SPINE_OVERLAY
          password 3 5c0c2e6087f1e976cc0f58d4a90b33e6
          update-source loopback1
          ebgp-multihop 2
          address-family l2vpn evpn
            send-community
            send-community extended
        neighbor 10.30.1.1
          inherit peer SPINE_OVERLAY
          description SPINE-1
        neighbor 10.30.1.2
          inherit peer SPINE_OVERLAY
          description SPINE-2
        neighbor 10.30.4.4
          inherit peer SPINE
          description SPINE-1
        neighbor 10.30.4.10
          inherit peer SPINE
          description SPINE-2
      evpn
        vni 10000 l2
          rd 10.30.1.13:100
          route-target import 10000:100
          route-target export 10000:100
        vni 20000 l2
          rd 10.30.1.13:200
          route-target import 20000:200
          route-target export 20000:200

###########################################################################################################

Результат настройки VxLAN EVPN L2:

      sh bgp l2vpn evpn summary
      sh bgp l2vpn evpn
      sh mac address-table
      sh ip arp suppression-cache detail

      Leaf-11# sh bgp l2vpn evpn summary
      BGP summary information for VRF default, address family L2VPN EVPN
      BGP router identifier 10.30.0.11, local AS number 65031
      BGP table version is 59, L2VPN EVPN config peers 2, capable peers 2
      9 network entries and 12 paths using 2196 bytes of memory
      BGP attribute entries [5/860], BGP AS path entries [1/10]
      BGP community entries [0/0], BGP clusterlist entries [0/0]
      
      Neighbor        V    AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
      10.30.1.1       4 65030     388     356       59    0    0 05:48:44 3         
      10.30.1.2       4 65030     389     357       59    0    0 05:49:05 3         
      
           
      Leaf-11# sh bgp l2vpn evpn
      BGP routing table information for VRF default, address family L2VPN EVPN
      BGP table version is 59, Local Router ID is 10.30.0.11
      Status: s-suppressed, x-deleted, S-stale, d-dampened, h-history, *-valid, >-best
      Path type: i-internal, e-external, c-confed, l-local, a-aggregate, r-redist, I-i
      njected
      Origin codes: i - IGP, e - EGP, ? - incomplete, | - multipath, & - backup, 2 - b
      est2
      
         Network            Next Hop            Metric     LocPrf     Weight Path
      Route Distinguisher: 10.30.1.11:100    (L2VNI 10000)
      *>l[2]:[0]:[0]:[48]:[0050.7966.6806]:[0]:[0.0.0.0]/216
                            10.30.1.11                        100      32768 i
      *>e[2]:[0]:[0]:[48]:[0050.7966.6808]:[0]:[0.0.0.0]/216
                            10.30.1.13                                     0 65030 65033 i
      *>l[2]:[0]:[0]:[48]:[0050.7966.6806]:[32]:[10.30.10.1]/248
                            10.30.1.11                        100      32768 i
      *>e[2]:[0]:[0]:[48]:[0050.7966.6808]:[32]:[10.30.10.11]/248
                            10.30.1.13                                     0 65030 65033 i
      *>l[3]:[0]:[32]:[10.30.1.11]/88
                            10.30.1.11                        100      32768 i
      *>e[3]:[0]:[32]:[10.30.1.13]/88
                            10.30.1.13                                     0 65030 65033 i
      
      Route Distinguisher: 10.30.1.13:100
      * e[2]:[0]:[0]:[48]:[0050.7966.6808]:[0]:[0.0.0.0]/216
                            10.30.1.13                                     0 65030 65033 i
      *>e                   10.30.1.13                                     0 65030 65033 i
      * e[2]:[0]:[0]:[48]:[0050.7966.6808]:[32]:[10.30.10.11]/248
                            10.30.1.13                                     0 65030 65033 i
      *>e                   10.30.1.13                                     0 65030 65033 i
      *>e[3]:[0]:[32]:[10.30.1.13]/88
                            10.30.1.13                                     0 65030 65033 i
      * e                   10.30.1.13                                     0 65030 65033 i
      
      
      Leaf-11# sh mac address-table
      Legend: 
              * - primary entry, G - Gateway MAC, (R) - Routed MAC, O - Overlay MAC
              age - seconds since last seen,+ - primary entry using vPC Peer-Link,
              (T) - True, (F) - False, C - ControlPlane MAC, ~ - vsan
         VLAN     MAC Address      Type      age     Secure NTFY Ports
      ---------+-----------------+--------+---------+------+----+------------------
      *   10     0050.7966.6806   dynamic  0         F      F    Eth1/7
      C   10     0050.7966.6808   dynamic  0         F      F    nve1(10.30.1.13)
      G    -     5003.0000.1b08   static   -         F      F    sup-eth1(R)
      
      
      Leaf-11# sh ip arp suppression-cache detail
      
      Flags: + - Adjacencies synced via CFSoE
             L - Local Adjacency
             R - Remote Adjacency
             L2 - Learnt over L2 interface
             PS - Added via L2RIB, Peer Sync
             RO - Dervied from L2RIB Peer Sync Entry
      
      Ip Address      Age      Mac Address    Vlan Physical-ifindex    Flags    Remote
       Vtep Addrs
      
      10.30.10.1      00:04:47 0050.7966.6806   10 Ethernet1/7         L2
      10.30.10.11     00:04:47 0050.7966.6808   10 (null)              R        10.30.1.13 

#####################
#####################

    Leaf-12# sh bgp l2vpn evpn summary
    sh bgp l2vpn evpnBGP summary information for VRF default, address family L2VPN EVPN
    BGP router identifier 10.30.0.12, local AS number 65032
    BGP table version is 61, L2VPN EVPN config peers 2, capable peers 2
    9 network entries and 12 paths using 2196 bytes of memory
    BGP attribute entries [5/860], BGP AS path entries [1/10]
    BGP community entries [0/0], BGP clusterlist entries [0/0]
    
    Neighbor        V    AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
    10.30.1.1       4 65030     391     360       61    0    0 05:52:40 3         
    10.30.1.2       4 65030     390     362       61    0    0 05:53:56 3         
    Leaf-12# sh bgp l2vpn evpn
    BGP routing table information for VRF default, address family L2VPN EVPN
    BGP table version is 61, Local Router ID is 10.30.0.12
    Status: s-suppressed, x-deleted, S-stale, d-dampened, h-history, *-valid, >-best
    Path type: i-internal, e-external, c-confed, l-local, a-aggregate, r-redist, I-i
    njected
    Origin codes: i - IGP, e - EGP, ? - incomplete, | - multipath, & - backup, 2 - b
    est2
    
       Network            Next Hop            Metric     LocPrf     Weight Path
    Route Distinguisher: 10.30.1.12:200    (L2VNI 20000)
    *>l[2]:[0]:[0]:[48]:[0050.7966.6807]:[0]:[0.0.0.0]/216
                          10.30.1.12                        100      32768 i
    *>e[2]:[0]:[0]:[48]:[0050.7966.6809]:[0]:[0.0.0.0]/216
                          10.30.1.13                                     0 65030 65033 i
    *>l[2]:[0]:[0]:[48]:[0050.7966.6807]:[32]:[10.30.20.1]/248
                          10.30.1.12                        100      32768 i
    *>e[2]:[0]:[0]:[48]:[0050.7966.6809]:[32]:[10.30.20.11]/248
                          10.30.1.13                                     0 65030 65033 i
    *>l[3]:[0]:[32]:[10.30.1.12]/88
                          10.30.1.12                        100      32768 i
    *>e[3]:[0]:[32]:[10.30.1.13]/88
                          10.30.1.13                                     0 65030 65033 i
    
    Route Distinguisher: 10.30.1.13:200
    * e[2]:[0]:[0]:[48]:[0050.7966.6809]:[0]:[0.0.0.0]/216
                          10.30.1.13                                     0 65030 65033 i
    *>e                   10.30.1.13                                     0 65030 65033 i
    * e[2]:[0]:[0]:[48]:[0050.7966.6809]:[32]:[10.30.20.11]/248
                          10.30.1.13                                     0 65030 65033 i
    *>e                   10.30.1.13                                     0 65030 65033 i
    *>e[3]:[0]:[32]:[10.30.1.13]/88
                          10.30.1.13                                     0 65030 65033 i
    * e                   10.30.1.13                                     0 65030 65033 i
    
    
    Leaf-12# sh mac address-table
    Legend: 
            * - primary entry, G - Gateway MAC, (R) - Routed MAC, O - Overlay MAC
            age - seconds since last seen,+ - primary entry using vPC Peer-Link,
            (T) - True, (F) - False, C - ControlPlane MAC, ~ - vsan
       VLAN     MAC Address      Type      age     Secure NTFY Ports
    ---------+-----------------+--------+---------+------+----+------------------
    *   20     0050.7966.6807   dynamic  0         F      F    Eth1/7
    C   20     0050.7966.6809   dynamic  0         F      F    nve1(10.30.1.13)
    G    -     5004.0000.1b08   static   -         F      F    sup-eth1(R)
    Leaf-12# sh ip arp suppression-cache detail
    
    Flags: + - Adjacencies synced via CFSoE
           L - Local Adjacency
           R - Remote Adjacency
           L2 - Learnt over L2 interface
           PS - Added via L2RIB, Peer Sync
           RO - Dervied from L2RIB Peer Sync Entry
    
    Ip Address      Age      Mac Address    Vlan Physical-ifindex    Flags    Remote
     Vtep Addrs
    
    10.30.20.1      00:00:57 0050.7966.6807   20 Ethernet1/7         L2
    10.30.20.11     00:07:05 0050.7966.6809   20 (null)              R        10.30.1.13  

#####################
#####################

    Leaf-13# sh bgp l2vpn evpn summary
    sh bgp l2vpn evpnBGP summary information for VRF default, address family L2VPN EVPN
    BGP router identifier 10.30.0.13, local AS number 65033
    BGP table version is 115, L2VPN EVPN config peers 2, capable peers 2
    18 network entries and 24 paths using 4392 bytes of memory
    BGP attribute entries [10/1720], BGP AS path entries [2/20]
    BGP community entries [0/0], BGP clusterlist entries [0/0]
    
    Neighbor        V    AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
    10.30.1.1       4 65030     385     363      115    0    0 05:56:04 6         
    10.30.1.2       4 65030     385     364      115    0    0 05:56:27 6         
    Leaf-13# sh bgp l2vpn evpn
    BGP routing table information for VRF default, address family L2VPN EVPN
    BGP table version is 115, Local Router ID is 10.30.0.13
    Status: s-suppressed, x-deleted, S-stale, d-dampened, h-history, *-valid, >-best
    Path type: i-internal, e-external, c-confed, l-local, a-aggregate, r-redist, I-i
    njected
    Origin codes: i - IGP, e - EGP, ? - incomplete, | - multipath, & - backup, 2 - b
    est2
    
       Network            Next Hop            Metric     LocPrf     Weight Path
    Route Distinguisher: 10.30.1.11:100
    * e[2]:[0]:[0]:[48]:[0050.7966.6806]:[0]:[0.0.0.0]/216
                          10.30.1.11                                     0 65030 65031 i
    *>e                   10.30.1.11                                     0 65030 65031 i
    * e[2]:[0]:[0]:[48]:[0050.7966.6806]:[32]:[10.30.10.1]/248
                          10.30.1.11                                     0 65030 65031 i
    *>e                   10.30.1.11                                     0 65030 65031 i
    *>e[3]:[0]:[32]:[10.30.1.11]/88
                          10.30.1.11                                     0 65030 65031 i
    * e                   10.30.1.11                                     0 65030 65031 i
    
    Route Distinguisher: 10.30.1.12:200
    *>e[2]:[0]:[0]:[48]:[0050.7966.6807]:[0]:[0.0.0.0]/216
                          10.30.1.12                                     0 65030 65032 i
    * e                   10.30.1.12                                     0 65030 65032 i
    * e[2]:[0]:[0]:[48]:[0050.7966.6807]:[32]:[10.30.20.1]/248
                          10.30.1.12                                     0 65030 65032 i
    *>e                   10.30.1.12                                     0 65030 65032 i
    *>e[3]:[0]:[32]:[10.30.1.12]/88
                          10.30.1.12                                     0 65030 65032 i
    * e                   10.30.1.12                                     0 65030 65032 i
    
    Route Distinguisher: 10.30.1.13:100    (L2VNI 10000)
    *>e[2]:[0]:[0]:[48]:[0050.7966.6806]:[0]:[0.0.0.0]/216
                          10.30.1.11                                     0 65030 65031 i
    *>l[2]:[0]:[0]:[48]:[0050.7966.6808]:[0]:[0.0.0.0]/216
                          10.30.1.13                        100      32768 i
    *>e[2]:[0]:[0]:[48]:[0050.7966.6806]:[32]:[10.30.10.1]/248
                          10.30.1.11                                     0 65030 65031 i
    *>l[2]:[0]:[0]:[48]:[0050.7966.6808]:[32]:[10.30.10.11]/248
                          10.30.1.13                        100      32768 i
    *>e[3]:[0]:[32]:[10.30.1.11]/88
                          10.30.1.11                                     0 65030 65031 i
    *>l[3]:[0]:[32]:[10.30.1.13]/88
                          10.30.1.13                        100      32768 i
    
    Route Distinguisher: 10.30.1.13:200    (L2VNI 20000)
    *>e[2]:[0]:[0]:[48]:[0050.7966.6807]:[0]:[0.0.0.0]/216
                          10.30.1.12                                     0 65030 65032 i
    *>l[2]:[0]:[0]:[48]:[0050.7966.6809]:[0]:[0.0.0.0]/216
                          10.30.1.13                        100      32768 i
    *>e[2]:[0]:[0]:[48]:[0050.7966.6807]:[32]:[10.30.20.1]/248
                          10.30.1.12                                     0 65030 65032 i
    *>l[2]:[0]:[0]:[48]:[0050.7966.6809]:[32]:[10.30.20.11]/248
                          10.30.1.13                        100      32768 i
    *>e[3]:[0]:[32]:[10.30.1.12]/88
                          10.30.1.12                                     0 65030 65032 i
    *>l[3]:[0]:[32]:[10.30.1.13]/88
                          10.30.1.13                        100      32768 i
    
    
    Leaf-13# sh mac address-table
    sh ip arp suppression-cache detailLegend: 
            * - primary entry, G - Gateway MAC, (R) - Routed MAC, O - Overlay MAC
            age - seconds since last seen,+ - primary entry using vPC Peer-Link,
            (T) - True, (F) - False, C - ControlPlane MAC, ~ - vsan
       VLAN     MAC Address      Type      age     Secure NTFY Ports
    ---------+-----------------+--------+---------+------+----+------------------
    C   10     0050.7966.6806   dynamic  0         F      F    nve1(10.30.1.11)
    *   10     0050.7966.6808   dynamic  0         F      F    Eth1/6
    C   20     0050.7966.6807   dynamic  0         F      F    nve1(10.30.1.12)
    *   20     0050.7966.6809   dynamic  0         F      F    Eth1/7
    G    -     5005.0000.1b08   static   -         F      F    sup-eth1(R)
    Leaf-13# sh ip arp suppression-cache detail
    
    Flags: + - Adjacencies synced via CFSoE
           L - Local Adjacency
           R - Remote Adjacency
           L2 - Learnt over L2 interface
           PS - Added via L2RIB, Peer Sync
           RO - Dervied from L2RIB Peer Sync Entry
    
    Ip Address      Age      Mac Address    Vlan Physical-ifindex    Flags    Remote
     Vtep Addrs
    
    10.30.10.11     00:09:51 0050.7966.6808   10 Ethernet1/6         L2
    10.30.10.1      00:09:52 0050.7966.6806   10 (null)              R        10.30.1.11  
    10.30.20.11     00:09:47 0050.7966.6809   20 Ethernet1/7         L2
    10.30.20.1      00:09:48 0050.7966.6807   20 (null)              R        10.30.1.12  

#####################
#####################

Проверка IP связности клиентов для Vlan_Id:10 и Vlan_Id:20

    Client-1> ping 10.30.10.11
    84 bytes from 10.30.10.11 icmp_seq=1 ttl=64 time=25.626 ms
    84 bytes from 10.30.10.11 icmp_seq=2 ttl=64 time=8.721 ms
    84 bytes from 10.30.10.11 icmp_seq=3 ttl=64 time=10.120 ms
    84 bytes from 10.30.10.11 icmp_seq=4 ttl=64 time=8.109 ms
    84 bytes from 10.30.10.11 icmp_seq=5 ttl=64 time=14.256 ms
    
    Client-2> ping 10.30.20.11
    84 bytes from 10.30.20.11 icmp_seq=1 ttl=64 time=12.247 ms
    84 bytes from 10.30.20.11 icmp_seq=2 ttl=64 time=8.695 ms
    84 bytes from 10.30.20.11 icmp_seq=3 ttl=64 time=14.371 ms
    84 bytes from 10.30.20.11 icmp_seq=4 ttl=64 time=12.476 ms
    84 bytes from 10.30.20.11 icmp_seq=5 ttl=64 time=19.167 ms
    
    Client-3> ping 10.30.10.1
    84 bytes from 10.30.10.1 icmp_seq=1 ttl=64 time=24.060 ms
    84 bytes from 10.30.10.1 icmp_seq=2 ttl=64 time=12.190 ms
    84 bytes from 10.30.10.1 icmp_seq=3 ttl=64 time=13.973 ms
    84 bytes from 10.30.10.1 icmp_seq=4 ttl=64 time=9.400 ms
    84 bytes from 10.30.10.1 icmp_seq=5 ttl=64 time=10.719 ms
    
    Client-4> ping 10.30.20.1
    84 bytes from 10.30.20.1 icmp_seq=1 ttl=64 time=19.664 ms
    84 bytes from 10.30.20.1 icmp_seq=2 ttl=64 time=8.544 ms
    84 bytes from 10.30.20.1 icmp_seq=3 ttl=64 time=12.592 ms
    84 bytes from 10.30.20.1 icmp_seq=4 ttl=64 time=12.637 ms
    84 bytes from 10.30.20.1 icmp_seq=5 ttl=64 time=9.134 ms

    
    
