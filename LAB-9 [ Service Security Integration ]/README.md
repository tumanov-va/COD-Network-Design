Защита проекта 

Тема: Внедрение сервисных устройств и МСЭ в фабрику

Цели проекта: 
 
       Настройка фабрики по Multi-POD архитектуре
       Внедрение сервисных устройств на примере МСЭ Cisco ASA
       Подготовка основы для внедрения схемы с независимыми HA парами или кластером МСЭ в фабрику с Multi-Site архитектурой.

Запланированные задачи в рамках реализации проекта: 

        1) Настройка фабрики по архитектуре Multi-POD
        2) Интеграция сервисных устройств в фабрику. Для внедрения выбраны устройства МСЭ - Cisco ASA, 
           образующие High-Availablility пару, разнесенные между PODами.
        3) Для всех клиентов доступно подключение во внешнюю сеть (IP: 8.8.8.8 / 5.5.5.5)
        4) Обеспечить взаимодействие IP-префиксов клиентов, принадлежащих разным тенантам (VRF) через HA пару МСЭ согласно матрице доступа.
           Иные взаимодействия запрещены.
        
Физическая схема
![Схема внедрения](https://github.com/tumanov-va/COD-Network-Design/assets/134439784/3fadd47d-9eed-49d8-8706-412302434418)


Адресное пространство для построения Underlay 

       POD-1:
       Loopback интерфейсы: 10.30.0.0/24 [ Lo0 ]
       P2P интерфейсы SPINE-LEAF: 10.30.4.0/24
       POD-2:
       Loopback интерфейсы: 10.30.100.0/24 [ Lo0 ]
       P2P интерфейсы SPINE-LEAF: 10.30.104.0/24     

Адресное пространство для построения Overlay:

       POD-1:
       Loopback интерфейсы: 10.30.1.0/24 [ Lo1 ]
       POD-2:
       Loopback интерфейсы: 10.30.101.0/24 [ Lo1 ]

Адресное пространство клиентов в тенантах:
  
      POD-1: 10.0.10.0/24  Vlan_Id: 10 (VRF PROM) 
      POD-1: 10.1.30.0/24  Vlan_Id: 30 (VRF TEST) 
      POD-2: 10.0.20.0/24  Vlan_Id: 20 (VRF PROM) 
      POD-2: 10.1.40.0/24  Vlan_Id: 40 (VRF TEST) 
      
Логическая схема интеграции МСЭ
![Логическая схема интеграции МСЭ](https://github.com/tumanov-va/COD-Network-Design/assets/134439784/3578b0d1-11a3-4d20-b327-73b298065c54)

Для упрощения восприятия информации о MAC адресах клиентов и ключевых устройств, параметры указаны статично в настройках:

      Client-1 # VRF-PROM # POD-1 # IP-10.0.10.10 # MAC-1000.1000.1010
      Client-1 # VRF-TEST # POD-1 # IP-10.1.30.10 # MAC-1000.1000.3010
      Client-2 # VRF-PROM # POD-2 # IP-10.0.20.10 # MAC-1000.1000.2010
      Client-2 # VRF-TEST # POD-2 # IP-10.1.40.10 # MAC-1000.1000.4010
      
      ASA-INSIDE # VRF-PROM # Primary node # 10.0.0.2 # MAC #  1000.000.0002
      ASA-INSIDE # VRF-PROM # Secondary node # 10.0.0.3 # MAC #  1000.000.0003
      ASA-OUTSIDE # VRF-PROM # Primary node # 10.0.0.10 # MAC #  1000.000.0010
      ASA-OUTSIDE # VRF-PROM # Secondary node # 10.0.0.11 # MAC #  1000.000.0011
      
      ASA-INSIDE # VRF-TEST # Primary node # 10.1.0.2 # MAC #  1001.000.0002
      ASA-INSIDE # VRF-TEST # Secondary node # 10.1.0.3 # MAC #  1001.000.0003
      ASA-OUTSIDE # VRF-TEST # Primary node # 10.1.0.10 # MAC #  1001.000.0010
      ASA-OUTSIDE # VRF-TEST # Secondary node # 10.1.0.11 # MAC #  1001.000.0011


Полная информация о конфигурации устройств и выводе диагностических команд представлены в следующих директория проекта:
    
        https://github.com/tumanov-va/COD-Network-Design/tree/1cfa72a7dc87710d8a3612c4292c5630f4f146ab/LAB-9%20%5B%20Service%20Security%20Integration%20%5D/Configs
        
        https://github.com/tumanov-va/COD-Network-Design/tree/1cfa72a7dc87710d8a3612c4292c5630f4f146ab/LAB-9%20%5B%20Service%20Security%20Integration%20%5D/AUDIT
                
        
        
      
    




