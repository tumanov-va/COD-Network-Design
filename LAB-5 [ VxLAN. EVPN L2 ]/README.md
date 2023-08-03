Цель:  Настроить Overlay на основе VxLAN EVPN для L2 связанности между клиентами

Описание: Настроить BGP peering между Leaf и Spine в AF l2vpn evpn

Настроена связанность между клиентами в первой зоне

Адресное пространство для построения Underlay 

      Loopback интерфейсы: 10.30.0.0/24 [ Lo0 ]
      P2P интерфейсы SPINE-LEAF: 10.30.4.0/24

Адресное пространство для построения Overlay:

      Loopback интерфейсы: 10.30.1.0/24 [ Lo1 ]
      Client1_Net: 10.30.10.0/24

Физическая схема
![Схема](https://github.com/tumanov-va/COD-Network-Design/assets/134439784/5b6056df-74e4-4162-b7ab-7685eea96fa5)


Приложения к лабораторной работе:

LAB-5 / [ VxLAN EVPN L2 Config].txt - Настройка устройств

LAB-5 / [ VxLAN EVPN L2 Output].txt - Результат настройки VxLAN EVPN L2
