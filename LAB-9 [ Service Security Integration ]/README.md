# Защита проекта 

# Тема: Внедрение сервисных устройств и МСЭ в фабрику

# Цели проекта: 
 
       Настройка фабрики по Multi-POD архитектуре
       Внедрение сервисных устройств на примере МСЭ Cisco ASA
       Подготовка основы для внедрения схемы с независимыми HA парами или кластером МСЭ в фабрику с Multi-Site архитектурой.

# Запланированные задачи в рамках реализации проекта: 

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

# Для упрощения восприятия информации о MAC адресах клиентов и ключевых устройств, параметры указаны статично в настройках:

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


# Полная информация о конфигурации устройств и выводе диагностических команд представлены в следующих директориях проекта:
    
        LAB-9 [ Service Security Integration ]/Configs
        https://github.com/tumanov-va/COD-Network-Design/tree/1cfa72a7dc87710d8a3612c4292c5630f4f146ab/LAB-9%20%5B%20Service%20Security%20Integration%20%5D/Configs
        
        LAB-9 [ Service Security Integration ]/AUDIT
        https://github.com/tumanov-va/COD-Network-Design/tree/1cfa72a7dc87710d8a3612c4292c5630f4f146ab/LAB-9%20%5B%20Service%20Security%20Integration%20%5D/AUDIT
                

# Разберём выполнение условий проекта на примере взаимодействия следующих устройств:

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


# Проверка доступов согласно поставленной задачи:

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


# Техническа часть проекта (Диагностика):

     LEAF-11# sh mac address-table dynamic vlan 10
     Legend: 
             * - primary entry, G - Gateway MAC, (R) - Routed MAC, O - Overlay MAC
             age - seconds since last seen,+ - primary entry using vPC Peer-Link,
             (T) - True, (F) - False, C - ControlPlane MAC, ~ - vsan
        VLAN     MAC Address      Type      age     Secure NTFY Ports
     ---------+-----------------+--------+---------+------+----+------------------
     *   10     1000.1000.1010   dynamic  0         F      F    Eth1/3


#L2FM - Layer2 Forwarding Manager:

     LEAF-11# sh system internal l2fm event-history debugs | in 1000.1000.1010
     2023 Oct 10 23:28:44.213690: E_DEBUG    l2fm [567]: l2fm_send_ntfn_to_am(17613): Sending old_index = 0x0, new_index = 0x1a000400 vlan: 10 mac: 1000.1000.1010 is_del: 0  
     2023 Oct 10 23:28:43.825217: E_DEBUG    l2fm [567]: l2fm_mac_regist_add_entry(6853): Adding node to delete registration database is_reg: 1 immed_notif: 1 MAC: 1000.1000.1010, ifindex: 0x901000a, phy ifindex: 0x1a000400  
     2023 Oct 10 21:56:30.907688: E_DEBUG    l2fm [567]: l2fm_l2rib_add_delete_local_mac_routes(2381): To L2RIB: topo-id: 10, macaddr: 1000.1000.1010, nhifindx: 0x1a000400 peer_addr 0x1a000400 
     2023 Oct 10 21:56:30.897485: E_DEBUG    l2fm [567]: l2fm_macdb_insert(8968): temp_str = slot 0 fe 0 mac 1000.1000.1010 vlan 10 flags 0x400107 hints 0 E8 NL lc  : if_index 0x1a000400 old_if_index 0
     2023 Oct 10 21:56:30.882918: E_DEBUG    l2fm [567]: l2fm_mcec_rmdb_delete(222): Deleting MAC 1000.1000.1010 vlan 10 from RMDB 

#L2RIB:

     LEAF-11# sh l2route evpn mac evi 10 
     Flags -(Rmac):Router MAC (Stt):Static (L):Local (R):Remote (V):vPC link 
     (Dup):Duplicate (Spl):Split (Rcv):Recv (AD):Auto-Delete (D):Del Pending
     (S):Stale (C):Clear, (Ps):Peer Sync (O):Re-Originated (Nho):NH-Override
     (Pf):Permanently-Frozen, (Orp): Orphan
     
     Topology    Mac Address    Prod   Flags         Seq No     Next-Hops                              
     ----------- -------------- ------ ------------- ---------- ---------------------------------------
     10          1000.1000.1010 Local  L,            0          Eth1/3                                
     
    LEAF-11# sh system internal l2rib event-history mac | include 1000.1000.1010
    2023 Oct 10 23:28:45.585308: E_DEBUG    l2rib [415]: [l2rib_obj_mac_bind_child:451] (10,1000.1000.1010): Bound MAC-IP: a0a:a:: to MAC, total MAC-IP linked count: 1 
    2023 Oct 10 21:56:31.048354: E_DEBUG    l2rib [415]: [l2rib_show_mac_rt:2535] (10,1000.1000.1010,3): NH[0]: Eth1/3 egressVNI: 0 
    2023 Oct 10 21:56:31.048309: E_DEBUG    l2rib [415]: [l2rib_show_mac_rt:2526] (10,1000.1000.1010,3): res: Regular ESI: (F) peerID: 0 NVE ifHandle: 1224736769 MH port-channel ifIndex: 0 nhCount: 1 encapType: 0 
    2023 Oct 10 21:56:31.048302: E_DEBUG    l2rib [415]: [l2rib_show_mac_rt:2516] (10,1000.1000.1010,3): VNI/EVI: 10000 rtFlags: L, adminDist: 6, seqNum: 0 ecmpLabel: 0 SOO: 0(--) 
    2023 Oct 10 21:56:31.047974: E_DEBUG    l2rib [415]: [l2rib_svr_mac_ent_gpb_encode:1124] (10,1000.1000.1010,3): Encoding MAC best route (ADD, client id 7) 
    2023 Oct 10 21:56:31.026449: E_DEBUG    l2rib [415]: [l2rib_obj_mac_route_create:3172] (10,1000.1000.1010,3):  SOO: 0, peerID: 0, PC ifIndex: 0 
    2023 Oct 10 21:56:31.026275: E_DEBUG    l2rib [415]: [l2rib_obj_mac_route_create:3163] (10,1000.1000.1010,3): MAC route created with seqNum: 0, flags: L, (),  
    2023 Oct 10 21:56:31.026214: E_DEBUG    l2rib [415]: [l2rib_obj_mac_route_create:3070] (10,1000.1000.1010,3): Route is local, isMacRemoteAtTheDelete: 0 
    2023 Oct 10 21:56:30.936114: E_DEBUG    l2rib [415]: [l2rib_client_show_route_msg:1412] Rcvd MAC ROUTE msg: (10, 1000.1000.1010), vni 0, admin_dist 0, seq 0, soo 0, 

# BGP L2VPN: 

    POD-1:  

    LEAF-11# sh bgp l2vpn evpn vni-id 10000
    BGP routing table information for VRF default, address family L2VPN EVPN
    BGP table version is 69, Local Router ID is 10.30.0.11
    Status: s-suppressed, x-deleted, S-stale, d-dampened, h-history, *-valid, >-best
    Path type: i-internal, e-external, c-confed, l-local, a-aggregate, r-redist, I-injected
    Origin codes: i - IGP, e - EGP, ? - incomplete, | - multipath, & - backup, 2 - best2
    
       Network            Next Hop            Metric     LocPrf     Weight Path
    Route Distinguisher: 10.30.0.11:32777    (L2VNI 10000)
    *>l[2]:[0]:[0]:[48]:[1000.1000.1010]:[0]:[0.0.0.0]/216
                          10.30.1.11                        100      32768 i
    *>l[2]:[0]:[0]:[48]:[1000.1000.1010]:[32]:[10.0.10.10]/272
                          10.30.1.11                        100      32768 i
    *>l[3]:[0]:[32]:[10.30.1.11]/88
                          10.30.1.11                        100      32768 i
                          

    LEAF-11# show bgp internal event-history events | in 1000.1000.1010
    2023 Oct 10 23:28:47.003037: E_DEBUG    bgp  [772]: (default) IMP: [L2VPN EVPN] Not importing 10.30.0.11:32777:[2]:[0]:[0]:[48]:[1000.1000.1010]:[32]:[10.0.10.10]/144 from RD 10.30.0.11:32777 to RD 10.30.0.11:32777: 
    source and target rdinfo are same 
    2023 Oct 10 23:28:47.002954: E_DEBUG    bgp  [772]: (default) IMP: [L2VPN EVPN] Import of 10.30.0.11:32777:[2]:[0]:[0]:[48]:[1000.1000.1010]:[32]:[10.0.10.10]/144 (EVI: 10000) to RD 10.30.0.11:65534 (0) inhibited, not importing local paths 
    2023 Oct 10 23:28:47.002947: E_DEBUG    bgp  [772]: (default) IMP: [L2VPN EVPN] Import of 10.30.0.11:32777:[2]:[0]:[0]:[48]:[1000.1000.1010]:[32]:[10.0.10.10]/144 (EVI: 10000) to RD 10.30.0.11:9910 (9910) inhibited, not importing local paths 
    2023 Oct 10 23:28:47.002930: E_DEBUG    bgp  [772]: (PROM) IMP: [IPv4 Unicast] Import of 10.30.0.11:32777:[2]:[0]:[0]:[48]:[1000.1000.1010]:[32]:[10.0.10.10]/144 (EVI: 10000) to RD 10.30.0.11:9910 (0) inhibited, not importing local paths 
    2023 Oct 10 23:28:47.000222: E_DEBUG    bgp  [772]: (default) RIB: [L2VPN EVPN] 10.30.0.11:32777:[2]:[0]:[0]:[48]:[1000.1000.1010]:[32]:[10.0.10.10]/144 is not in rib, no del 
    2023 Oct 10 23:28:47.000150: E_DEBUG    bgp  [772]: (default) RIB: [L2VPN EVPN] For 10.30.0.11:32777:[2]:[0]:[0]:[48]:[1000.1000.1010]:[32]:[10.0.10.10]/144, added 0 next hops, suppress 0 
    2023 Oct 10 23:28:47.000142: E_DEBUG    bgp  [772]: (default) RIB: [L2VPN EVPN] 10.30.0.11:32777:[2]:[0]:[0]:[48]:[1000.1000.1010]:[32]:[10.0.10.10]/144 via 10.30.1.11 is local, no add 
    2023 Oct 10 23:28:46.998657: E_DEBUG    bgp  [772]: (default) RIB: [L2VPN EVPN] Add/delete 10.30.0.11:32777:[2]:[0]:[0]:[48]:[1000.1000.1010]:[32]:[10.0.10.10]/144, flags=0x100, in_rib: no 
    2023 Oct 10 23:28:46.283687: E_DEBUG    bgp  [772]: (default) RIB: [L2VPN EVPN] add prefix 10.30.0.11:32777:[2]:[0]:[0]:[48]:[1000.1000.1010]:[32]:[10.0.10.10] (flags 0x1) : OK, total 3 
    2023 Oct 10 23:28:46.136493: E_DEBUG    bgp  [772]: EVT: Received from L2RIB MAC-IP route: Add ESI 0000.0000.0000.0000.0000 EVI 0/10000 mac 1000.1000.1010 ip 10.0.10.10 L3 VNI 9910 flags 0x000001 soo 0 seq 0, reorig :0 vrfid 0 
    2023 Oct 10 21:56:31.082437: E_DEBUG    bgp  [772]: (default) IMP: [L2VPN EVPN] Not importing 10.30.0.11:32777:[2]:[0]:[0]:[48]:[1000.1000.1010]:[0]:[0.0.0.0]/112 from RD 10.30.0.11:32777 to RD 10.30.0.11:32777: 
    source and target rdinfo are same 
    2023 Oct 10 21:56:31.082372: E_DEBUG    bgp  [772]: (default) IMP: [L2VPN EVPN] Import of 10.30.0.11:32777:[2]:[0]:[0]:[48]:[1000.1000.1010]:[0]:[0.0.0.0]/112 (EVI: 10000) to RD 10.30.0.11:65534 (0) inhibited, not importing local paths 
    2023 Oct 10 21:56:31.081243: E_DEBUG    bgp  [772]: (default) RIB: [L2VPN EVPN] 10.30.0.11:32777:[2]:[0]:[0]:[48]:[1000.1000.1010]:[0]:[0.0.0.0]/112 is not in rib, no del 
    2023 Oct 10 21:56:31.080684: E_DEBUG    bgp  [772]: (default) RIB: [L2VPN EVPN] For 10.30.0.11:32777:[2]:[0]:[0]:[48]:[1000.1000.1010]:[0]:[0.0.0.0]/112, added 0 next hops, suppress 0 
    2023 Oct 10 21:56:31.080678: E_DEBUG    bgp  [772]: (default) RIB: [L2VPN EVPN] 10.30.0.11:32777:[2]:[0]:[0]:[48]:[1000.1000.1010]:[0]:[0.0.0.0]/112 via 10.30.1.11 is local, no add 
    2023 Oct 10 21:56:31.080290: E_DEBUG    bgp  [772]: (default) RIB: [L2VPN EVPN] Add/delete 10.30.0.11:32777:[2]:[0]:[0]:[48]:[1000.1000.1010]:[0]:[0.0.0.0]/112, flags=0x100, in_rib: no 
    2023 Oct 10 21:56:31.073227: E_DEBUG    bgp  [772]: (default) RIB: [L2VPN EVPN] add prefix 10.30.0.11:32777:[2]:[0]:[0]:[48]:[1000.1000.1010]:[0]:[0.0.0.0] (flags 0x1) : OK, total 2 
    2023 Oct 10 21:56:31.063781: E_DEBUG    bgp  [772]: EVT: Received from L2RIB MAC route: Add ESI 0000.0000.0000.0000.0000 EVI 0/10000 mac 1000.1000.1010 flags 0x000002 soo 0 seq 0 reorig: 0 
        

    LEAF-11# sh bgp l2vpn evpn 1000.1000.1010
    BGP routing table information for VRF default, address family L2VPN EVPN
    Route Distinguisher: 10.30.0.11:32777    (L2VNI 10000)
    BGP routing table entry for [2]:[0]:[0]:[48]:[1000.1000.1010]:[0]:[0.0.0.0]/216, version 7
    Paths: (1 available, best #1)
    Flags: (0x000102) (high32 00000000) on xmit-list, is not in l2rib/evpn
    
      Advertised path-id 1
      Path type: local, path is valid, is best path, no labeled nexthop
      AS-Path: NONE, path locally originated
        10.30.1.11 (metric 0) from 0.0.0.0 (10.30.0.11)
          Origin IGP, MED not set, localpref 100, weight 32768
          Received label 10000
          Extcommunity: RT:65031:10000 ENCAP:8
    
      Path-id 1 advertised to peers:
        10.30.1.1          10.30.1.2      
    BGP routing table entry for [2]:[0]:[0]:[48]:[1000.1000.1010]:[32]:[10.0.10.10]/272, version 61
    Paths: (1 available, best #1)
    Flags: (0x000102) (high32 00000000) on xmit-list, is not in l2rib/evpn
    
      Advertised path-id 1
      Path type: local, path is valid, is best path, no labeled nexthop
      AS-Path: NONE, path locally originated
        10.30.1.11 (metric 0) from 0.0.0.0 (10.30.0.11)
          Origin IGP, MED not set, localpref 100, weight 32768
          Received label 10000 9910
          Extcommunity: RT:65030:9910 RT:65031:10000 ENCAP:8 Router MAC:5008.0000.1b08
    
      Path-id 1 advertised to peers:
        10.30.1.1          10.30.1.2      
    
    
    BL-11# sh bgp l2vpn evpn 1000.1000.1010
    BGP routing table information for VRF default, address family L2VPN EVPN
    Route Distinguisher: 10.30.0.11:32777
    BGP routing table entry for [2]:[0]:[0]:[48]:[1000.1000.1010]:[32]:[10.0.10.10]/272, version 707
    Paths: (2 available, best #1)
    Flags: (0x000202) (high32 00000000) on xmit-list, is not in l2rib/evpn, is not in HW
    
      Advertised path-id 1
      Path type: internal, path is valid, is best path, no labeled nexthop
                 Imported to 2 destination(s)
                 Imported paths list: PROM L3-9910
      AS-Path: NONE, path sourced internal to AS
        10.30.1.11 (metric 0) from 10.30.1.1 (10.30.0.1)
          Origin IGP, MED not set, localpref 100, weight 0
          Received label 10000 9910
          Extcommunity: RT:65030:9910 RT:65031:10000 ENCAP:8 Router MAC:5008.0000.1b08
          Originator: 10.30.0.11 Cluster list: 10.30.0.1 
    
      Path type: internal, path is valid, not best reason: Neighbor Address, no labeled nexthop
      AS-Path: NONE, path sourced internal to AS
        10.30.1.11 (metric 0) from 10.30.1.2 (10.30.0.2)
          Origin IGP, MED not set, localpref 100, weight 0
          Received label 10000 9910
          Extcommunity: RT:65030:9910 RT:65031:10000 ENCAP:8 Router MAC:5008.0000.1b08
          Originator: 10.30.0.11 Cluster list: 10.30.0.2 
    
      Path-id 1 not advertised to any peer
    
    Route Distinguisher: 10.30.0.10:9910    (L3VNI 9910)
    BGP routing table entry for [2]:[0]:[0]:[48]:[1000.1000.1010]:[32]:[10.0.10.10]/272, version 708
    Paths: (1 available, best #1)
    Flags: (0x000202) (high32 00000000) on xmit-list, is not in l2rib/evpn, is not in HW
    
      Advertised path-id 1
      Path type: internal, path is valid, is best path, no labeled nexthop
                 Imported from 10.30.0.11:32777:[2]:[0]:[0]:[48]:[1000.1000.1010]:[32]:[10.0.10.10]/272 
      AS-Path: NONE, path sourced internal to AS
        10.30.1.11 (metric 0) from 10.30.1.1 (10.30.0.1)
          Origin IGP, MED not set, localpref 100, weight 0
          Received label 10000 9910
          Extcommunity: RT:65030:9910 RT:65031:10000 ENCAP:8 Router MAC:5008.0000.1b08
          Originator: 10.30.0.11 Cluster list: 10.30.0.1 
    
      Path-id 1 not advertised to any peer
      
 POD-2:  Проверка на устройстве подключения резервной ноды МСЭ BL-21:

    BL-21# sh bgp l2vpn evpn 1000.1000.1010
    BGP routing table information for VRF default, address family L2VPN EVPN
    Route Distinguisher: 10.30.0.11:32777
    BGP routing table entry for [2]:[0]:[0]:[48]:[1000.1000.1010]:[32]:[10.0.10.10]/272, version 1246
    Paths: (2 available, best #2)
    Flags: (0x000202) (high32 00000000) on xmit-list, is not in l2rib/evpn, is not in HW
    
      Path type: internal, path is valid, not best reason: Router Id, no labeled nexthop
      AS-Path: 65031 , path sourced external to AS
        10.30.1.11 (metric 0) from 10.30.101.2 (10.30.100.2)
          Origin IGP, MED not set, localpref 100, weight 0
          Received label 10000 9910
          Extcommunity: RT:65030:9910 RT:65031:10000 ENCAP:8 Router MAC:5008.0000.1b08
    
      Advertised path-id 1
      Path type: internal, path is valid, is best path, no labeled nexthop
                 Imported to 2 destination(s)
                 Imported paths list: PROM L3-9910
      AS-Path: 65031 , path sourced external to AS
        10.30.1.11 (metric 0) from 10.30.101.1 (10.30.100.1)
          Origin IGP, MED not set, localpref 100, weight 0
          Received label 10000 9910
          Extcommunity: RT:65030:9910 RT:65031:10000 ENCAP:8 Router MAC:5008.0000.1b08
    
      Path-id 1 not advertised to any peer
    
    Route Distinguisher: 10.30.100.10:9910    (L3VNI 9910)
    BGP routing table entry for [2]:[0]:[0]:[48]:[1000.1000.1010]:[32]:[10.0.10.10]/272, version 1245
    Paths: (1 available, best #1)
    Flags: (0x000202) (high32 00000000) on xmit-list, is not in l2rib/evpn, is not in HW
    
      Advertised path-id 1
      Path type: internal, path is valid, is best path, no labeled nexthop
                 Imported from 10.30.0.11:32777:[2]:[0]:[0]:[48]:[1000.1000.1010]:[32]:[10.0.10.10]/272 
      AS-Path: 65031 , path sourced external to AS
        10.30.1.11 (metric 0) from 10.30.101.1 (10.30.100.1)
          Origin IGP, MED not set, localpref 100, weight 0
          Received label 10000 9910
          Extcommunity: RT:65030:9910 RT:65031:10000 ENCAP:8 Router MAC:5008.0000.1b08
    
      Path-id 1 not advertised to any peer

 Взаимодействие клиентов, находящихся в разных VRF, происходит через МСЭ по правилу Default route

    BL-11# sh ip route 10.0.10.10 vrf PROM
    IP Route Table for VRF "PROM"
    '*' denotes best ucast next-hop
    '**' denotes best mcast next-hop
    '[x/y]' denotes [preference/metric]
    '%<string>' in via output denotes VRF <string>
    
    10.0.10.10/32, ubest/mbest: 1/0
        *via 10.30.1.11%default, [200/0], 01:30:36, bgp-65031, internal, tag 65031, segid: 9910 tunnelid: 0xa1e010b encap: VXLAN
     
    BL-11# sh ip route 10.1.40.10 vrf PROM
    IP Route Table for VRF "PROM"
    '*' denotes best ucast next-hop
    '**' denotes best mcast next-hop
    '[x/y]' denotes [preference/metric]
    '%<string>' in via output denotes VRF <string>
    
    0.0.0.0/0, ubest/mbest: 1/0
        *via 10.0.0.2, [1/0], 01:37:31, static, tag 9910
        

     BL-11# sh ip route 10.1.40.10 vrf TEST
     IP Route Table for VRF "TEST"
     '*' denotes best ucast next-hop
     '**' denotes best mcast next-hop
     '[x/y]' denotes [preference/metric]
     '%<string>' in via output denotes VRF <string>
     
     10.1.40.10/32, ubest/mbest: 1/0
         *via 10.30.101.11%default, [200/0], 01:27:30, bgp-65031, internal, tag 65032, segid: 9920 tunnelid: 0xa1e650b encap: VXLAN
      
     BL-11# sh ip route 10.0.10.10 vrf TEST
     IP Route Table for VRF "TEST"
     '*' denotes best ucast next-hop
     '**' denotes best mcast next-hop
     '[x/y]' denotes [preference/metric]
     '%<string>' in via output denotes VRF <string>
     
     0.0.0.0/0, ubest/mbest: 1/0
         *via 10.1.0.2, [1/0], 01:39:23, static, tag 9920


 



