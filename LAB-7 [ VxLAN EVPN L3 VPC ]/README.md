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

  

