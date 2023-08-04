Цель: Настроить отказоустойчивое подключение клиентов с использованием VPC

Описание: Подключение клиентов 2-я линками к различным Leaf. Настройка агрегированных каналов со стороны клиентов. Настройка VPC для работы в Overlay сети.

Физическая схема
![Схема](https://github.com/tumanov-va/COD-Network-Design/assets/134439784/99f6610c-d91f-43c6-b4e4-0edeef09f05a)
        
Адресное пространство для построения Underlay 

      Loopback интерфейсы: 10.30.0.0/24 [ Lo0 ]
      P2P интерфейсы SPINE-LEAF: 10.30.4.0/24

Адресное пространство для построения Overlay:

      Loopback интерфейсы: 10.30.1.0/24 [ Lo1 ]
      Customer1_Net: 10.30.10.0/24
      Customer1_Vlan_Id: 10 (PROM) L3 Сервис
      Customer2_Net: 10.30.20.0/24
      Customer2_Vlan_Id: 20 (TEST) L2 Сервис
      Customer3_Net: 10.30.20.0/24
      Customer3_Vlan_Id: 30 (PSI) L3 Сервис

Приложения к лабораторной работе:

    LAB-7 / [ VxLAN EVPN L3 VPC Config].txt - Настройка устройств
    LAB-7 / [ VxLAN EVPN L3 VPC Output].txt - Результат настройки VxLAN EVPN L3 VPC
    LAB-7 / Схема.png - Физическая схема
    
###########################################################################################################
Пример конфигурации для коммутаторов Spine и Leaf:

      Spine's ASn = 65030
      Leaf11 ASn = 65031
      Leaf12 ASn = 65031
      Leaf13 ASn = 65033
      Leaf14 ASn = 65034

        hostname Spine-1
        ip host Spine-1 10.30.0.1
        ip host Spine-2 10.30.0.2
        ip host Leaf-11 10.30.0.11
        ip host Leaf-12 10.30.0.12
        ip host Leaf-13 10.30.0.13
        ip host Leaf-14 10.30.0.14
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
            
        interface loopback0
          description Router_ID # ControlPlane_EVPN
          ip address 10.30.0.1/32
        
        interface loopback1
          description Router_ID # DataPlane_VxLAN
          ip address 10.30.1.1/32
        
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

###########
###########

        hostname Leaf-11
        
        cfs eth distribute
        nv overlay evpn
        feature bgp
        feature fabric forwarding
        feature interface-vlan
        feature vn-segment-vlan-based
        feature lacp
        feature vpc
        feature bfd
        clock timezone MSK 3 0
        feature nv overlay
                
        ip host Spine-1 10.30.0.1
        ip host Spine-2 10.30.0.2
        ip host Leaf-11 10.30.0.11
        ip host Leaf-12 10.30.0.12
        ip host Leaf-13 10.30.0.13
        ip host Leaf-14 10.30.0.14
        
        fabric forwarding anycast-gateway-mac 0001.0001.9999
        vlan 1,10,13,20,30
        vlan 10
          name PROM
          vn-segment 10000
        vlan 13
          name L3VNI_Transit_SVI
          vn-segment 13000
        vlan 20
          name TEST
          vn-segment 20000
        vlan 30
          name PSI
          vn-segment 30000
        
        spanning-tree vlan 1-3967 priority 4096
        
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
        vrf context FAILOVER
        vrf context L3VNI_Transit
          vni 13000
          rd 10.30.1.11:13000
          address-family ipv4 unicast
            route-target import 65031:13000 evpn
            route-target import 65032:13000 evpn
            route-target import 65033:13000 evpn
            route-target import 65034:13000 evpn
            route-target export 65031:13000 evpn
        
        hardware access-list tcam region racl 512
        hardware access-list tcam region arp-ether 256 double-wide
        
        vpc domain 10
          peer-switch
          role priority 5
          peer-keepalive destination 1.1.1.2 source 1.1.1.1 vrf FAILOVER
          delay restore 10
          peer-gateway
          ip arp synchronize
        
        interface Vlan10
          description SVI_PROM_Clients-GW
          no shutdown
          vrf member L3VNI_Transit
          no ip redirects
          ip address 10.30.10.254/24
          no ipv6 redirects
          fabric forwarding mode anycast-gateway
        
        interface Vlan13
          description L3VNI_Transit_SVI # W/O IP ADDRESS #
          no shutdown
          vrf member L3VNI_Transit
          no ip redirects
          ip forward
          no ipv6 redirects
        
        interface Vlan30
          description SVI_PSI_Clients-GW
          no shutdown
          vrf member L3VNI_Transit
          no ip redirects
          ip address 10.30.30.254/24
          no ipv6 redirects
          fabric forwarding mode anycast-gateway
        
        interface port-channel2
          description # CLIENT-1 #
          switchport mode trunk
          vpc 10
        
        interface port-channel100
          description # VPC-FAILOVER-LINK #
          no switchport
          vrf member FAILOVER
          ip address 1.1.1.1/30
        
        interface port-channel200
          description # VPC-PEER-LINK #
          switchport mode trunk
          spanning-tree port type network
          vpc peer-link
        
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
          ip address 10.30.1.253/32 secondary
        
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
          neighbor 10.30.4.8
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

###########
###########

        hostname Leaf-12
        
        cfs eth distribute
        nv overlay evpn
        feature bgp
        feature fabric forwarding
        feature interface-vlan
        feature vn-segment-vlan-based
        feature lacp
        feature vpc
        feature bfd
        clock timezone MSK 3 0
        feature nv overlay
        
        ip host Spine-1 10.30.0.1
        ip host Spine-2 10.30.0.2
        ip host Leaf-11 10.30.0.11
        ip host Leaf-12 10.30.0.12
        ip host Leaf-13 10.30.0.13
        ip host Leaf-14 10.30.0.14
        
        fabric forwarding anycast-gateway-mac 0001.0001.9999
        vlan 1,10,13,20,30
        vlan 10
          name PROM
          vn-segment 10000
        vlan 13
          name L3VNI_Transit_SVI
          vn-segment 13000
        vlan 20
          name TEST
          vn-segment 20000
        vlan 30
          name PSI
          vn-segment 30000
        
        spanning-tree vlan 1-3967 priority 8192
        
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
        vrf context FAILOVER
        vrf context L3VNI_Transit
          vni 13000
          rd 10.30.1.12:13000
          address-family ipv4 unicast
            route-target import 65031:13000 evpn
            route-target import 65032:13000 evpn
            route-target import 65033:13000 evpn
            route-target import 65034:13000 evpn
            route-target export 65032:13000 evpn
        
        hardware access-list tcam region racl 512
        hardware access-list tcam region arp-ether 256 double-wide
        vpc domain 10
          peer-switch
          role priority 10
          peer-keepalive destination 1.1.1.1 source 1.1.1.2 vrf FAILOVER
          delay restore 10
          peer-gateway
          ip arp synchronize
        
        interface Vlan10
          description SVI_PROM_Clients-GW
          no shutdown
          vrf member L3VNI_Transit
          no ip redirects
          ip address 10.30.10.254/24
          no ipv6 redirects
          fabric forwarding mode anycast-gateway
        
        interface Vlan13
          description L3VNI_Transit_SVI # W/O IP ADDRESS #
          no shutdown
          vrf member L3VNI_Transit
          no ip redirects
          ip forward
          no ipv6 redirects
        
        interface Vlan30
          description SVI_PSI_Clients-GW
          no shutdown
          vrf member L3VNI_Transit
          no ip redirects
          ip address 10.30.30.254/24
          no ipv6 redirects
          fabric forwarding mode anycast-gateway
        
        interface port-channel2
          description # CLIENT-1 #
          switchport mode trunk
          vpc 10
        
        interface port-channel100
          description # VPC-FAILOVER-LINK #
          no switchport
          vrf member FAILOVER
          ip address 1.1.1.2/30
        
        interface port-channel200
          description # VPC-PEER-LINK #
          switchport mode trunk
          spanning-tree port type network
          vpc peer-link
        
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
          ip address 10.30.1.253/32 secondary
        
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
          neighbor 10.30.4.2
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
        
        


