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

                                         
