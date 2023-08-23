Цель: 

        Настройка EVPN для передачи маршрутной информации через type-5 анонсы.

Физическая схема
![Схема](https://github.com/tumanov-va/COD-Network-Design/assets/134439784/73125bfe-2b54-4c10-a772-47ff2b507e68)

Адресное пространство для построения Underlay 

      Loopback интерфейсы: 10.30.0.0/24 [ Lo0 ]
      P2P интерфейсы SPINE-LEAF: 10.30.4.0/24

Адресное пространство для построения Overlay:

      Loopback интерфейсы: 10.30.1.0/24 [ Lo1 ]
      Customer1_Net: 10.30.10.0/24
      Customer1_Vlan_Id: 10 (PROM) L3 Сервис
      Customer2_Net: 10.30.20.0/24
      Customer2_Vlan_Id: 20 (TEST) L3 Сервис
      Customer3_Net: 10.30.30.0/24
      Customer3_Vlan_Id: 30 (PSI) L3 Сервис
      Customer4_Net: 10.30.40.0/24
      Customer4_Vlan_Id: 40 (DEV) L3 Сервис

![изображение](https://github.com/tumanov-va/COD-Network-Design/assets/134439784/a91b118a-bd89-4e6a-9401-16cb58ab1026)
