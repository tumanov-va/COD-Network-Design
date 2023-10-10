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
    
        LAB-9 [ Service Security Integration ]/Configs
        https://github.com/tumanov-va/COD-Network-Design/tree/1cfa72a7dc87710d8a3612c4292c5630f4f146ab/LAB-9%20%5B%20Service%20Security%20Integration%20%5D/Configs
        
        LAB-9 [ Service Security Integration ]/AUDIT
        https://github.com/tumanov-va/COD-Network-Design/tree/1cfa72a7dc87710d8a3612c4292c5630f4f146ab/LAB-9%20%5B%20Service%20Security%20Integration%20%5D/AUDIT
                

Разберём выполнение условий проекта на примере взаимодействия следующих устройств:

     Client-1 # VRF-PROM # POD-1 # IP-10.0.10.10 # MAC-1000.1000.1010
     Client-2 # VRF-TEST # POD-2 # IP-10.1.40.10 # MAC-1000.1000.4010

Дополнительно проверим выполнение поставленной задачи при переключении с основной на резервную ноду МСЭ Cisco ASA (Failover).

Обработка периметрового и межзонного трафика на основной ноде МСЭ:

         FW-ASA-PRI/pri/act# sh failover state  
         
                        State          Last Failure Reason      Date/Time
         This host  -   Primary
                        Active         Ifc Failure              07:14:34 UTC Oct 10 2023
                                       PROM inside: Failed
                                       PROM outside: Failed
                                       TEST inside: Failed
                                       TEST outside: Failed
         Other host -   Secondary
                        Standby Ready  None
         
         ====Configuration State===
                 Sync Done
                 Sync Done - STANDBY
         ====Communication State===
                 Mac set


Проверка доступов согласно поставленной задачи:

    1) Для всех клиентов доступно подключение во внешнюю сеть (IP: 8.8.8.8 / 5.5.5.5)

      Client1-PROM#ping 8.8.8.8   
      Type escape sequence to abort.
      Sending 5, 100-byte ICMP Echos to 8.8.8.8, timeout is 2 seconds:
      !!!!!
      Success rate is 100 percent (5/5), round-trip min/avg/max = 16/18/23 ms

    2) Обеспечить взаимодействие IP-префиксов клиентов, принадлежащих разным тенантам (VRF) через HA пару МСЭ согласно матрице доступа.
      
      Client1-PROM#ping 10.1.40.10
      Type escape sequence to abort.
      Sending 5, 100-byte ICMP Echos to 10.1.40.10, timeout is 2 seconds:
      !!!!!
      Success rate is 100 percent (5/5), round-trip min/avg/max = 31/36/45 ms

      Иные взаимодействия запрещены.
      
      Client1-PROM#ping 10.1.30.10
      Type escape sequence to abort.
      Sending 5, 100-byte ICMP Echos to 10.1.30.10, timeout is 2 seconds:
      .....
      Success rate is 0 percent (0/5)

Дополнительная проверка взаимодействия клиентов различных тенантов (трафик между зонами обрабатывается правилами внедрённого сервисного устройства - МСЭ):

      Client1-PROM#traceroute 10.1.40.10
      Type escape sequence to abort.
      Tracing the route to 10.1.40.10
      VRF info: (vrf in name/id, vrf out name/id)
        1 10.0.10.254 8 msec 4 msec 2 msec
        2 10.0.0.1 20 msec 19 msec 12 msec
        3 10.0.0.2 31 msec *  34 msec
        4 10.0.0.12 22 msec 20 msec 18 msec
        5 10.1.0.10 19 msec *  29 msec
        6 10.1.0.1 24 msec 23 msec 20 msec
        7 10.1.40.254 51 msec 43 msec 35 msec
        8 10.1.40.10 50 msec *  40 msec
   
       Client1-PROM#traceroute 10.1.30.10
       Type escape sequence to abort.
       Tracing the route to 10.1.30.10
       VRF info: (vrf in name/id, vrf out name/id)
         1 10.0.10.254 8 msec 1 msec 2 msec
         2 10.0.0.1 12 msec 9 msec 15 msec
         3 10.0.0.2 16 msec *  18 msec
         4 10.0.0.12 11 msec 11 msec 14 msec
         5  *  *  * 
         6  *  *  * 
         7  *  *  * 
         8  *  *  * 
         ----------
        29  *  *  * 
        30  *  *  * 


Техническа часть проекта (Диагностика):

     LEAF-11# sh mac address-table dynamic vlan 10
     Legend: 
             * - primary entry, G - Gateway MAC, (R) - Routed MAC, O - Overlay MAC
             age - seconds since last seen,+ - primary entry using vPC Peer-Link,
             (T) - True, (F) - False, C - ControlPlane MAC, ~ - vsan
        VLAN     MAC Address      Type      age     Secure NTFY Ports
     ---------+-----------------+--------+---------+------+----+------------------
     *   10     1000.1000.1010   dynamic  0         F      F    Eth1/3


# L2FM - Layer2 Forwarding Manager:
     LEAF-11# sh system internal l2fm event-history debugs | in 1000.1000.1010
     2023 Oct 10 23:28:44.213690: E_DEBUG    l2fm [567]: l2fm_send_ntfn_to_am(17613): Sending old_index = 0x0, new_index = 0x1a000400 vlan: 10 mac: 1000.1000.1010 is_del: 0  
     2023 Oct 10 23:28:43.825217: E_DEBUG    l2fm [567]: l2fm_mac_regist_add_entry(6853): Adding node to delete registration database is_reg: 1 immed_notif: 1 MAC: 1000.1000.1010, ifindex: 0x901000a, phy ifindex: 0x1a000400  
     2023 Oct 10 21:56:30.907688: E_DEBUG    l2fm [567]: l2fm_l2rib_add_delete_local_mac_routes(2381): To L2RIB: topo-id: 10, macaddr: 1000.1000.1010, nhifindx: 0x1a000400 peer_addr 0x1a000400 
     2023 Oct 10 21:56:30.897485: E_DEBUG    l2fm [567]: l2fm_macdb_insert(8968): temp_str = slot 0 fe 0 mac 1000.1000.1010 vlan 10 flags 0x400107 hints 0 E8 NL lc  : if_index 0x1a000400 old_if_index 0
     2023 Oct 10 21:56:30.882918: E_DEBUG    l2fm [567]: l2fm_mcec_rmdb_delete(222): Deleting MAC 1000.1000.1010 vlan 10 from RMDB 


     



        
        
      
    




