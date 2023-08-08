Цель:  Настройка каждого клиента в своем VNI. 

Описание: VxLAN EVPN для L3

Настройка маршрутизацию между клиентами

Адресное пространство для построения Underlay 

      Loopback интерфейсы: 10.30.0.0/24 [ Lo0 ]
      P2P интерфейсы SPINE-LEAF: 10.30.4.0/24

Адресное пространство для построения Overlay:

      Loopback интерфейсы: 10.30.1.0/24 [ Lo1 ]
      
      Client1_Net: 10.30.10.0/24
      Client1_Vlan_Id: 10 (PROM) L3 Сервис
      
      Client2_Net: 10.30.20.0/24
      Client2_Vlan_Id: 20 (TEST) L2 Сервис

      Client3_Net: 10.30.30.0/24
      Client3_Vlan_Id: 30 (PSI) L3 Сервис

Физическая схема
![Схема](https://github.com/tumanov-va/COD-Network-Design/assets/134439784/cf5ed9a6-2819-4e4b-b1af-3a66a0fab898)

Приложения к лабораторной работе:

     LAB-6 / [ VxLAN EVPN L3 Config].txt - Настройка устройств
     LAB-6 / [ VxLAN EVPN L3 Output].txt - Результат настройки VxLAN EVPN L3
     LAB-6 / Схема.png - Физическая схема


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
      ipv6 address use-link-local-only
      no shutdown
    
    interface Ethernet1/2
      description # Leaf-12 # 10.30.0.12 # Port # Ethernet1/1
      no switchport
      mtu 9216
      no ip redirects
      ip address 10.30.4.2/31
      ipv6 address use-link-local-only
      no shutdown
    
    interface Ethernet1/3
      description # Leaf-13 # 10.30.0.13 # Port # Ethernet1/1
      no switchport
      mtu 9216
      no ip redirects
      ip address 10.30.4.4/31
      ipv6 address use-link-local-only
      no shutdown
    
    interface loopback0
      description Router_ID # ControlPlane_EVPN
      ip address 10.30.0.1/32
    
    interface loopback1
      description Router_ID # DataPlane_VxLAN
      ip address 10.30.1.1/32
    
    line console
      exec-timeout 60
    line vty
    boot nxos bootflash:/nxos64-cs.10.3.1.F.bin 
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
        bfd
        description LEAF_OVERLAY
        password 3 5c0c2e6087f1e976cc0f58d4a90b33e6
        update-source loopback1
        ebgp-multihop 2
        timers 3 15
        address-family l2vpn evpn
          send-community
          send-community extended
          route-map next-hop-unchanged out
      neighbor 10.30.4.0/24 remote-as route-map AS_FILTER
        bfd
        description LEAF
        password 3 5c0c2e6087f1e976cc0f58d4a90b33e6
        timers 3 15
        address-family ipv4 unicast

###############
###############

    hostname Leaf-11
    
    nv overlay evpn
    feature bgp
    feature fabric forwarding
    feature interface-vlan
    feature vn-segment-vlan-based
    feature bfd
    feature nv overlay
    
    hardware access-list tcam region racl 512
    hardware access-list tcam region arp-ether 256 double-wide
    
    fabric forwarding anycast-gateway-mac 0001.0001.9999
    vlan 1,10,13,20,30
    vlan 10
      name PROM
      vn-segment 10000
    vlan 13
      vn-segment 13000
    vlan 20
      name TEST
      vn-segment 20000
    vlan 30
      name PSI
      vn-segment 30000
    
    no cdp enable
    ip prefix-list PL_CONNECTED_OUT seq 10 permit 10.30.10.0/24 
    ip prefix-list PL_CONNECTED_OUT seq 20 permit 10.30.30.0/24 
    route-map REDISTRIBUTE_CONNECTED permit 10
      match interface loopback0 loopback1 
    route-map RM_EXPORT_CONNECTED permit 10
      match ip address prefix-list PL_CONNECTED_OUT 
    key chain UNDERLAY_KeyChain
      key 1
        key-string 7 073f007f7d3e363733
        cryptographic-algorithm HMAC-SHA-256
    vrf context L3VNI_Transit
      vni 13000
      rd 10.30.1.11:13000
      address-family ipv4 unicast
        route-target import 65031:13000 evpn
        route-target import 65032:13000 evpn
        route-target import 65033:13000 evpn
        route-target export 65031:13000 evpn
    
    interface Vlan10
      description SVI_PROM_Clients-GW
      no shutdown
      vrf member L3VNI_Transit
      ip address 10.30.10.254/24
      fabric forwarding mode anycast-gateway
    
    interface Vlan13
      description L3VNI_Transit_SVI # W/O IP ADDRESS #
      no shutdown
      vrf member L3VNI_Transit
      no ip redirects
      ip forward
    
    interface Vlan30
      description SVI_PSI_Clients-GW
      no shutdown
      vrf member L3VNI_Transit
      ip address 10.30.30.254/24
      fabric forwarding mode anycast-gateway
    
    interface nve1
      no shutdown
      host-reachability protocol bgp
      source-interface loopback1
      member vni 10000
        suppress-arp
        ingress-replication protocol bgp
      member vni 13000 associate-vrf
      member vni 20000
        suppress-arp
        ingress-replication protocol bgp
      member vni 30000
        suppress-arp
        ingress-replication protocol bgp
    
    interface loopback0
      description Router_ID # ControlPlane_EVPN
      ip address 10.30.0.11/32
    
    interface loopback1
      description Router_ID # DataPlane_VxLAN
      ip address 10.30.1.11/32
    icam monitor scale
    
    boot nxos bootflash:/nxos64-cs.10.3.1.F.bin 
    router bgp 65031
      router-id 10.30.0.11
      bestpath as-path multipath-relax
      log-neighbor-changes
      address-family ipv4 unicast
        redistribute direct route-map REDISTRIBUTE_CONNECTED
      address-family l2vpn evpn
      template peer SPINE
        bfd
        remote-as 65030
        description SPINE_UNDERLAY
        password 3 5c0c2e6087f1e976cc0f58d4a90b33e6
        timers 3 15
        address-family ipv4 unicast
      template peer SPINE_OVERLAY
        bfd
        remote-as 65030
        description SPINE_OVERLAY
        password 3 5c0c2e6087f1e976cc0f58d4a90b33e6
        update-source loopback1
        ebgp-multihop 2
        timers 3 15
        address-family l2vpn evpn
          send-community
          send-community extended
      neighbor 10.30.1.1
        inherit peer SPINE_OVERLAY
        description SPINE-1
      neighbor 10.30.1.2
        inherit peer SPINE_OVERLAY
        description SPINE-2
      neighbor 10.30.4.0
        inherit peer SPINE
        description SPINE-1
      neighbor 10.30.4.6
        inherit peer SPINE
        description SPINE-2
      vrf L3VNI_Transit
        address-family ipv4 unicast
          redistribute direct route-map RM_EXPORT_CONNECTED
    evpn
      vni 10000 l2
        rd 10.30.1.11:10
        route-target import 10000:10
        route-target export 10000:10
      vni 20000 l2
        rd 10.30.1.11:20
        route-target import 20000:20
        route-target export 20000:20
      vni 30000 l2
        rd 10.30.1.11:30
        route-target import 30000:30
        route-target export 30000:30

###############
###############
    
    hostname Leaf-12
    
    nv overlay evpn
    feature bgp
    feature fabric forwarding
    feature interface-vlan
    feature vn-segment-vlan-based
    feature bfd
    feature nv overlay
    
    hardware access-list tcam region racl 512
    hardware access-list tcam region arp-ether 256 double-wide
    
    fabric forwarding anycast-gateway-mac 0001.0001.9999
    vlan 1,10,13,20,30
    vlan 10
      name PROM
      vn-segment 10000
    vlan 13
      vn-segment 13000
    vlan 20
      name TEST
      vn-segment 20000
    vlan 30
      name PSI
      vn-segment 30000
    
    no cdp enable
    ip prefix-list PL_CONNECTED_OUT seq 10 permit 10.30.10.0/24 
    ip prefix-list PL_CONNECTED_OUT seq 20 permit 10.30.30.0/24 
    route-map REDISTRIBUTE_CONNECTED permit 10
      match interface loopback0 loopback1 
    route-map RM_EXPORT_CONNECTED permit 10
      match ip address prefix-list PL_CONNECTED_OUT 
    key chain UNDERLAY_KeyChain
      key 1
        key-string 7 073f007f7d3e363733
        cryptographic-algorithm HMAC-SHA-256
    vrf context L3VNI_Transit
      vni 13000
      rd 10.30.1.12:13000
      address-family ipv4 unicast
        route-target import 65031:13000 evpn
        route-target import 65032:13000 evpn
        route-target import 65033:13000 evpn
        route-target export 65032:13000 evpn
    vrf context management
    
    interface Vlan10
      description SVI_PROM_Clients-GW
      no shutdown
      vrf member L3VNI_Transit
      ip address 10.30.10.254/24
      fabric forwarding mode anycast-gateway
    
    interface Vlan13
      description L3VNI_Transit_SVI # W/O IP ADDRESS #
      no shutdown
      vrf member L3VNI_Transit
      no ip redirects
      ip forward
    
    interface Vlan30
      description SVI_PSI_Clients-GW
      no shutdown
      vrf member L3VNI_Transit
      ip address 10.30.30.254/24
      fabric forwarding mode anycast-gateway
    
    interface nve1
      no shutdown
      host-reachability protocol bgp
      source-interface loopback1
      member vni 10000
        suppress-arp
        ingress-replication protocol bgp
      member vni 13000 associate-vrf
      member vni 20000
        suppress-arp
        ingress-replication protocol bgp
      member vni 30000
        suppress-arp
        ingress-replication protocol bgp
    
    interface loopback0
      description Router_ID # ControlPlane_EVPN
      ip address 10.30.0.12/32
    
    interface loopback1
      description Router_ID # DataPlane_VxLAN
      ip address 10.30.1.12/32
    
    boot nxos bootflash:/nxos64-cs.10.3.1.F.bin 
    router bgp 65032
      router-id 10.30.0.12
      bestpath as-path multipath-relax
      log-neighbor-changes
      address-family ipv4 unicast
        redistribute direct route-map REDISTRIBUTE_CONNECTED
      address-family l2vpn evpn
      template peer SPINE
        bfd
        remote-as 65030
        description LEAF
        password 3 5c0c2e6087f1e976cc0f58d4a90b33e6
        timers 3 15
        address-family ipv4 unicast
      template peer SPINE_OVERLAY
        bfd
        remote-as 65030
        description SPINE_OVERLAY
        password 3 5c0c2e6087f1e976cc0f58d4a90b33e6
        update-source loopback1
        ebgp-multihop 2
        timers 3 15
        address-family l2vpn evpn
          send-community
          send-community extended
      neighbor 10.30.1.1
        inherit peer SPINE_OVERLAY
        description SPINE-1
      neighbor 10.30.1.2
        inherit peer SPINE_OVERLAY
        description SPINE-2
      neighbor 10.30.4.2
        inherit peer SPINE
        description SPINE-1
      neighbor 10.30.4.8
        inherit peer SPINE
        description SPINE-2
      vrf L3VNI_Transit
        address-family ipv4 unicast
          redistribute direct route-map RM_EXPORT_CONNECTED
    evpn
      vni 10000 l2
        rd 10.30.1.12:10
        route-target import 10000:10
        route-target export 10000:10
      vni 20000 l2
        rd 10.30.1.12:20
        route-target import 20000:20
        route-target export 20000:20
      vni 30000 l2
        rd 10.30.1.12:30
        route-target import 30000:30
        route-target export 30000:30

###############
###############

    hostname Leaf-13
    
    nv overlay evpn
    feature bgp
    feature fabric forwarding
    feature interface-vlan
    feature vn-segment-vlan-based
    feature bfd
    feature nv overlay
    
    hardware access-list tcam region racl 512
    hardware access-list tcam region arp-ether 256 double-wide
    
    fabric forwarding anycast-gateway-mac 0001.0001.9999
    vlan 1,10,13,20,30
    vlan 10
      name PROM
      vn-segment 10000
    vlan 13
      vn-segment 13000
    vlan 20
      name TEST
      vn-segment 20000
    vlan 30
      name PSI
      vn-segment 30000
    
    no cdp enable
    ip prefix-list PL_CONNECTED_OUT seq 10 permit 10.30.10.0/24 
    ip prefix-list PL_CONNECTED_OUT seq 20 permit 10.30.30.0/24 
    route-map REDISTRIBUTE_CONNECTED permit 10
      match interface loopback0 loopback1 
    route-map RM_EXPORT_CONNECTED permit 10
      match ip address prefix-list PL_CONNECTED_OUT 
    key chain UNDERLAY_KeyChain
      key 1
        key-string 7 073f007f7d3e363733
        cryptographic-algorithm HMAC-SHA-256
    vrf context L3VNI_Transit
      vni 13000
      rd 10.30.1.13:13000
      address-family ipv4 unicast
        route-target import 65031:13000 evpn
        route-target import 65032:13000 evpn
        route-target import 65033:13000 evpn
        route-target export 65033:13000 evpn
    
    interface Vlan10
      description SVI_PROM_Clients-GW
      no shutdown
      vrf member L3VNI_Transit
      ip address 10.30.10.254/24
      fabric forwarding mode anycast-gateway
    
    interface Vlan13
      description L3VNI_Transit_SVI # W/O IP ADDRESS #
      no shutdown
      vrf member L3VNI_Transit
      no ip redirects
      ip forward
    
    interface Vlan30
      description SVI_PSI_Clients-GW
      no shutdown
      vrf member L3VNI_Transit
      ip address 10.30.30.254/24
      fabric forwarding mode anycast-gateway
    
    interface nve1
      no shutdown
      host-reachability protocol bgp
      source-interface loopback1
      member vni 10000
        suppress-arp
        ingress-replication protocol bgp
      member vni 13000 associate-vrf
      member vni 20000
        suppress-arp
        ingress-replication protocol bgp
      member vni 30000
        suppress-arp
        ingress-replication protocol bgp
    
    interface loopback0
      description Router_ID # ControlPlane_EVPN
      ip address 10.30.0.13/32
    
    interface loopback1
      description Router_ID # DataPlane_VxLAN
      ip address 10.30.1.13/32
    
    boot nxos bootflash:/nxos64-cs.10.3.1.F.bin 
    router bgp 65033
      router-id 10.30.0.13
      bestpath as-path multipath-relax
      log-neighbor-changes
      address-family ipv4 unicast
        redistribute direct route-map REDISTRIBUTE_CONNECTED
      template peer SPINE
        bfd
        remote-as 65030
        description LEAF
        password 3 5c0c2e6087f1e976cc0f58d4a90b33e6
        timers 3 15
        address-family ipv4 unicast
      template peer SPINE_OVERLAY
        bfd
        remote-as 65030
        description SPINE_OVERLAY
        password 3 5c0c2e6087f1e976cc0f58d4a90b33e6
        update-source loopback1
        ebgp-multihop 2
        timers 3 15
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
      vrf L3VNI_Transit
        address-family ipv4 unicast
          redistribute direct route-map RM_EXPORT_CONNECTED
    evpn
      vni 10000 l2
        rd 10.30.1.13:10
        route-target import 10000:10
        route-target export 10000:10
      vni 20000 l2
        rd 10.30.1.13:20
        route-target import 20000:20
        route-target export 20000:20
      vni 30000 l2
        rd 10.30.1.13:30
        route-target import 30000:30
        route-target export 30000:30


###########################################################################################################

      Результат настройки VxLAN EVPN L3: (Полный вывод в файле LAB-6/[ VxLAN EVPN L3 Output ].txt)
      
      Leaf-12# sh nve peers
      Interface Peer-IP                                 State LearnType Uptime   Route
      r-Mac       
      --------- --------------------------------------  ----- --------- -------- -----
      ------------
      nve1      10.30.1.11                              Up    CP        07:00:55 5003.0000.1b08   
      nve1      10.30.1.13                              Up    CP        07:00:55 5005.0000.1b08   
      
      Leaf-12# sh ip route hmm vrf L3VNI_Transit
      sh running-config bgp
      IP Route Table for VRF "L3VNI_Transit"
      '*' denotes best ucast next-hop
      '**' denotes best mcast next-hop
      '[x/y]' denotes [preference/metric]
      '%<string>' in via output denotes VRF <string>
      
      10.30.10.2/32, ubest/mbest: 1/0, attached
          *via 10.30.10.2, Vlan10, [190/0], 00:33:51, hmm
      10.30.30.2/32, ubest/mbest: 1/0, attached
          *via 10.30.30.2, Vlan30, [190/0], 00:27:16, hmm
      
      
      Leaf-12# sh bgp l2vpn evpn summary
      sh bgp l2vpn evpn
      
      BGP summary information for VRF default, address family L2VPN EVPN
      BGP router identifier 10.30.0.12, local AS number 65032
      BGP table version is 257, L2VPN EVPN config peers 2, capable peers 2
      55 network entries and 81 paths using 15180 bytes of memory
      BGP attribute entries [45/16200], BGP AS path entries [2/20]
      BGP community entries [0/0], BGP clusterlist entries [0/0]
      
      Neighbor        V    AS    MsgRcvd    MsgSent   TblVer  InQ OutQ Up/Down  State/
      PfxRcd
      10.30.1.1       4 65030       8479       8434      257    0    0 06:42:10 22    
          
      10.30.1.2       4 65030       8486       8445      257    0    0 07:02:30 22    
          
      Neighbor        T    AS PfxRcd     Type-2     Type-3     Type-4     Type-5    
      10.30.1.1       I 65030 22         12         6          0          4         
      10.30.1.2       I 65030 22         12         6          0          4         


      Leaf-12# sh bgp l2vpn evpn
      BGP routing table information for VRF default, address family L2VPN EVPN
      BGP table version is 257, Local Router ID is 10.30.0.12
      Status: s-suppressed, x-deleted, S-stale, d-dampened, h-history, *-valid, >-best
      Path type: i-internal, e-external, c-confed, l-local, a-aggregate, r-redist, I-i
      njected
      Origin codes: i - IGP, e - EGP, ? - incomplete, | - multipath, & - backup, 2 - b
      est2
      
         Network            Next Hop            Metric     LocPrf     Weight Path
      Route Distinguisher: 10.30.1.11:10
      * e[2]:[0]:[0]:[48]:[0050.7966.6806]:[0]:[0.0.0.0]/216
                            10.30.1.11                                     0 65030 65031 i
      *>e                   10.30.1.11                                     0 65030 65031 i
      * e[2]:[0]:[0]:[48]:[0050.7966.6806]:[32]:[10.30.10.1]/272
                            10.30.1.11                                     0 65030 65031 i
      *>e                   10.30.1.11                                     0 65030 65031 i
      *>e[3]:[0]:[32]:[10.30.1.11]/88
                            10.30.1.11                                     0 65030 65031 i
      * e                   10.30.1.11                                     0 65030 65031 i
      
      Route Distinguisher: 10.30.1.11:20
      * e[2]:[0]:[0]:[48]:[0050.7966.680a]:[0]:[0.0.0.0]/216
                            10.30.1.11                                     0 65030 65031 i
      *>e                   10.30.1.11                                     0 65030 65031 i
      * e[2]:[0]:[0]:[48]:[0050.7966.680a]:[32]:[10.30.20.1]/248
                            10.30.1.11                                     0 65030 65031 i
      *>e                   10.30.1.11                                     0 65030 65031 i
      *>e[3]:[0]:[32]:[10.30.1.11]/88
                            10.30.1.11                                     0 65030 65031 i
      * e                   10.30.1.11                                     0 65030 65031 i
      
      Route Distinguisher: 10.30.1.11:30
      * e[2]:[0]:[0]:[48]:[0050.7966.680d]:[0]:[0.0.0.0]/216
                            10.30.1.11                                     0 65030 65031 i
      *>e                   10.30.1.11                                     0 65030 65031 i
      * e[2]:[0]:[0]:[48]:[0050.7966.680d]:[32]:[10.30.30.1]/272
                            10.30.1.11                                     0 65030 65031 i
      *>e                   10.30.1.11                                     0 65030 65031 i
      *>e[3]:[0]:[32]:[10.30.1.11]/88
                            10.30.1.11                                     0 65030 65031 i
      * e                   10.30.1.11                                     0 65030 65031 i
      
      Route Distinguisher: 10.30.1.11:13000
      *>e[5]:[0]:[0]:[24]:[10.30.10.0]/224
                            10.30.1.11                                     0 65030 65031 ?
      * e                   10.30.1.11                                     0 65030 65031 ?
      *>e[5]:[0]:[0]:[24]:[10.30.30.0]/224
                            10.30.1.11                                     0 65030 65031 ?
      * e                   10.30.1.11                                     0 65030 65031 ?
      
      Route Distinguisher: 10.30.1.12:10    (L2VNI 10000)
      *>e[2]:[0]:[0]:[48]:[0050.7966.6806]:[0]:[0.0.0.0]/216
                            10.30.1.11                                     0 65030 65031 i
      *>l[2]:[0]:[0]:[48]:[0050.7966.6807]:[0]:[0.0.0.0]/216
                            10.30.1.12                        100      32768 i
      *>e[2]:[0]:[0]:[48]:[0050.7966.6809]:[0]:[0.0.0.0]/216
                            10.30.1.13                                     0 65030 65033 i
      *>e[2]:[0]:[0]:[48]:[0050.7966.6806]:[32]:[10.30.10.1]/272
                            10.30.1.11                                     0 65030 65031 i
      *>l[2]:[0]:[0]:[48]:[0050.7966.6807]:[32]:[10.30.10.2]/272
                            10.30.1.12                        100      32768 i
      *>e[2]:[0]:[0]:[48]:[0050.7966.6809]:[32]:[10.30.10.3]/272
                            10.30.1.13                                     0 65030 65033 i
      *>e[3]:[0]:[32]:[10.30.1.11]/88
                            10.30.1.11                                     0 65030 65031 i
      *>l[3]:[0]:[32]:[10.30.1.12]/88
                            10.30.1.12                        100      32768 i
      *>e[3]:[0]:[32]:[10.30.1.13]/88
                            10.30.1.13                                     0 65030 65033 i
      
      Route Distinguisher: 10.30.1.12:20    (L2VNI 20000)
      *>e[2]:[0]:[0]:[48]:[0050.7966.6808]:[0]:[0.0.0.0]/216
                            10.30.1.13                                     0 65030 65033 i
      *>e[2]:[0]:[0]:[48]:[0050.7966.680a]:[0]:[0.0.0.0]/216
                            10.30.1.11                                     0 65030 65031 i
      *>l[2]:[0]:[0]:[48]:[0050.7966.680b]:[0]:[0.0.0.0]/216
                            10.30.1.12                        100      32768 i
      *>e[2]:[0]:[0]:[48]:[0050.7966.6808]:[32]:[10.30.20.3]/248
                            10.30.1.13                                     0 65030 65033 i
      *>e[2]:[0]:[0]:[48]:[0050.7966.680a]:[32]:[10.30.20.1]/248
                            10.30.1.11                                     0 65030 65031 i
      *>l[2]:[0]:[0]:[48]:[0050.7966.680b]:[32]:[10.30.20.2]/248
                            10.30.1.12                        100      32768 i
      *>e[3]:[0]:[32]:[10.30.1.11]/88
                            10.30.1.11                                     0 65030 65031 i
      *>l[3]:[0]:[32]:[10.30.1.12]/88
                            10.30.1.12                        100      32768 i
      *>e[3]:[0]:[32]:[10.30.1.13]/88
                            10.30.1.13                                     0 65030 65033 i
      
      Route Distinguisher: 10.30.1.12:30    (L2VNI 30000)
      *>e[2]:[0]:[0]:[48]:[0050.7966.680c]:[0]:[0.0.0.0]/216
                            10.30.1.13                                     0 65030 65033 i
      *>e[2]:[0]:[0]:[48]:[0050.7966.680d]:[0]:[0.0.0.0]/216
                            10.30.1.11                                     0 65030 65031 i
      *>l[2]:[0]:[0]:[48]:[0050.7966.680e]:[0]:[0.0.0.0]/216
                            10.30.1.12                        100      32768 i
      *>e[2]:[0]:[0]:[48]:[0050.7966.680c]:[32]:[10.30.30.3]/272
                            10.30.1.13                                     0 65030 65033 i
      *>e[2]:[0]:[0]:[48]:[0050.7966.680d]:[32]:[10.30.30.1]/272
                            10.30.1.11                                     0 65030 65031 i
      *>l[2]:[0]:[0]:[48]:[0050.7966.680e]:[32]:[10.30.30.2]/272
                            10.30.1.12                        100      32768 i
      *>e[3]:[0]:[32]:[10.30.1.11]/88
                            10.30.1.11                                     0 65030 65031 i
      *>l[3]:[0]:[32]:[10.30.1.12]/88
                            10.30.1.12                        100      32768 i
      *>e[3]:[0]:[32]:[10.30.1.13]/88
                            10.30.1.13                                     0 65030 65033 i
      
      Route Distinguisher: 10.30.1.13:10
      *>e[2]:[0]:[0]:[48]:[0050.7966.6809]:[0]:[0.0.0.0]/216
                            10.30.1.13                                     0 65030 65033 i
      * e                   10.30.1.13                                     0 65030 65033 i
      * e[2]:[0]:[0]:[48]:[0050.7966.6809]:[32]:[10.30.10.3]/272
                            10.30.1.13                                     0 65030 65033 i
      *>e                   10.30.1.13                                     0 65030 65033 i
      * e[3]:[0]:[32]:[10.30.1.13]/88
                            10.30.1.13                                     0 65030 65033 i
      *>e                   10.30.1.13                                     0 65030 65033 i
      
      Route Distinguisher: 10.30.1.13:20
      *>e[2]:[0]:[0]:[48]:[0050.7966.6808]:[0]:[0.0.0.0]/216
                            10.30.1.13                                     0 65030 65033 i
      * e                   10.30.1.13                                     0 65030 65033 i
      * e[2]:[0]:[0]:[48]:[0050.7966.6808]:[32]:[10.30.20.3]/248
                            10.30.1.13                                     0 65030 65033 i
      *>e                   10.30.1.13                                     0 65030 65033 i
      * e[3]:[0]:[32]:[10.30.1.13]/88
                            10.30.1.13                                     0 65030 65033 i
      *>e                   10.30.1.13                                     0 65030 65033 i
      
      Route Distinguisher: 10.30.1.13:30
      *>e[2]:[0]:[0]:[48]:[0050.7966.680c]:[0]:[0.0.0.0]/216
                            10.30.1.13                                     0 65030 65033 i
      * e                   10.30.1.13                                     0 65030 65033 i
      * e[2]:[0]:[0]:[48]:[0050.7966.680c]:[32]:[10.30.30.3]/272
                            10.30.1.13                                     0 65030 65033 i
      *>e                   10.30.1.13                                     0 65030 65033 i
      * e[3]:[0]:[32]:[10.30.1.13]/88
                            10.30.1.13                                     0 65030 65033 i
      *>e                   10.30.1.13                                     0 65030 65033 i
      
      Route Distinguisher: 10.30.1.13:13000
      * e[5]:[0]:[0]:[24]:[10.30.10.0]/224
                            10.30.1.13                                     0 65030 65033 ?
      *>e                   10.30.1.13                                     0 65030 65033 ?
      * e[5]:[0]:[0]:[24]:[10.30.30.0]/224
                            10.30.1.13                                     0 65030 65033 ?
      *>e                   10.30.1.13                                     0 65030 65033 ?
      
      Route Distinguisher: 10.30.1.12:13000    (L3VNI 13000)
      *>e[2]:[0]:[0]:[48]:[0050.7966.6806]:[32]:[10.30.10.1]/272
                            10.30.1.11                                     0 65030 65031 i
      *>e[2]:[0]:[0]:[48]:[0050.7966.6809]:[32]:[10.30.10.3]/272
                            10.30.1.13                                     0 65030 65033 i
      *>e[2]:[0]:[0]:[48]:[0050.7966.680c]:[32]:[10.30.30.3]/272
                            10.30.1.13                                     0 65030 65033 i
      *>e[2]:[0]:[0]:[48]:[0050.7966.680d]:[32]:[10.30.30.1]/272
                            10.30.1.11                                     0 65030 65031 i
      * e[5]:[0]:[0]:[24]:[10.30.10.0]/224
                            10.30.1.11                                     0 65030 65031 ?
      * e                   10.30.1.13                                     0 65030 65033 ?
      *>l                   10.30.1.12               0        100      32768 ?
      * e[5]:[0]:[0]:[24]:[10.30.30.0]/224
                            10.30.1.11                                     0 65030 65031 ?
      * e                   10.30.1.13                                     0 65030 65033 ?
      *>l                   10.30.1.12               0        100      32768 ?
      

