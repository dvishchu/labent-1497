Task TS01: H1(172.16.101.10) cannot ping H3(172.16.101.11) over vxlan via vlan101
=================================================================================

.. image:: assets/cfg01_topology.png
    :align: center

H1 node 

.. code-block:: console
    :linenos:
    :emphasize-lines: 4,11
    :class: highlight-command highlight-command-13 emphasize-hll

    ts01-H1#ping vrf h1 172.16.101.11
    Type escape sequence to abort.
    Sending 5, 100-byte ICMP Echos to 172.16.101.11, timeout is 2 seconds:
    .....
    Success rate is 0 percent (0/5)

    ts01-H1#sh ip arp vrf h1
    Protocol  Address          Age (min)  Hardware Addr   Type   Interface
    Internet  172.16.101.1            0   aabb.cc80.0300  ARPA   Vlan101
    Internet  172.16.101.10           -   0000.0001.0101  ARPA   Vlan101
    Internet  172.16.101.11           0   Incomplete      ARPA   

We will start with the leaf to which the host is connected – Leaf1. In the host information we saw that ARP is ``incomplete``. Lets check the same on Leaf1 and also look into the NVE peering.

.. note::

    Note that VRF “green” is used on the Leaf1 node for L3VNI. 

L1 node 

.. code-block:: console
    :linenos:
    :class: highlight-command

    ts01-L1#sh ip arp vrf green 
    Protocol  Address          Age (min)  Hardware Addr   Type   Interface
    Internet  10.1.254.3              -   aabb.cc80.0300  ARPA   Vlan901
    Internet  172.16.101.1            -   aabb.cc80.0300  ARPA   Vlan101
    Internet  172.16.101.10           1   0000.0001.0101  ARPA   Vlan101
    Internet  172.16.102.10           2   0000.0001.0102  ARPA   Vlan102

We won’t see ARPs for clients over remote VTEP

.. code-block:: console
    :linenos:
    :class: highlight-command

    ts01-L1#sh nve peers 
    'M' - MAC entry download flag  'A' - Adjacency download flag
    '4' - IPv4 flag  '6' - IPv6 flag

    Interface  VNI      Type Peer-IP          RMAC/Num_RTs   eVNI     state flags UP time
    nve1       50901    L3CP 10.1.254.4       aabb.cc80.0400 50901      UP  A/M/4 00:02:06
    nve1       50901    L3CP 10.1.254.5       aabb.cc80.0500 50901      UP  A/M/4 00:02:06
    nve1       50901    L3CP 10.1.254.6       aabb.cc80.0600 50901      UP  A/M/4 00:02:06
    nve1       50901    L3CP 10.1.254.7       aabb.cc80.0700 50901      UP  A/M/4 00:02:06
    nve1       10102    L2CP 10.1.254.4       4              10102      UP   N/A  00:02:06
    nve1       10102    L2CP 10.1.254.5       4              10102      UP   N/A  00:02:06
    nve1       10102    L2CP 10.1.254.6       2              10102      UP   N/A  00:02:06
    nve1       10102    L2CP 10.1.254.7       3              10102      UP   N/A  00:02:06

In the NVE peers table above that there are no entries that would be showing a peering over VNI ``10101``. Therefore, lets check the EVI for the vlan where we have H1 attached – vlan 101. 

The EVI outputs show the vlan 101 is mapped to the L2 VNI ``10110`` but the VTEP IP is ``UNKNOWN``.

.. code-block:: console
    :linenos:
    :emphasize-lines: 4,28,30
    :class: highlight-command highlight-command-11 emphasize-hll

    ts01-L1#sh l2vpn evpn evi vlan 101
    EVI   VLAN  Ether Tag  L2 VNI    Multicast     Pseudoport
    ----- ----- ---------- --------- ------------- ------------------
    101   101   0          10110     UNKNOWN       Et0/0:101 

    ts01-L1#sh l2vpn evpn evi vlan 101 detail 
    EVPN instance:       101 (VLAN Based)
    RD:                10.1.255.3:101 (auto)
    Import-RTs:        65001:101 
    Export-RTs:        65001:101 
    Per-EVI Label:     none
    State:             Established
    Replication Type:  Ingress (global)
    Encapsulation:     vxlan
    IP Local Learn:    Enabled (global)
    Adv. Def. Gateway: Enabled (global)
    Re-originate RT5:  Disabled
    Adv. Multicast:    Disabled (global)
    Vlan:              101
        Ethernet-Tag:    0
        State:           Established
        Flood Suppress:  Attached
        Core If:         
        Access If:       
        NVE If:          
        RMAC:            0000.0000.0000
        Core Vlan:       0
        L2 VNI:          10110  
        L3 VNI:          0
        VTEP IP:         UNKNOWN 
        Pseudoports:
        Ethernet0/0 service instance 101
            Routes: 1 MAC, 1 MAC/IP
        Peers:
        10.1.254.4
            Routes: 2 MAC, 2 MAC/IP, 1 IMET, 0 EAD
        10.1.254.5
            Routes: 2 MAC, 2 MAC/IP, 1 IMET, 0 EAD
        10.1.254.6
            Routes: 1 MAC, 1 MAC/IP, 1 IMET, 0 EAD
        10.1.254.7
            Routes: 1 MAC, 2 MAC/IP, 1 IMET, 0 EAD 

The MAC/IP information from BGP routes shows that the next show information is actually expecting ``10101``.

.. code-block:: console
    :linenos:
    :class: highlight-command

    ts01-L1#sh l2route evpn mac ip 
    EVI       ETag  Prod    Mac Address         Host IP                Next Hop(s)
    ----- ---------- ----- -------------- --------------- --------------------------
    101          0 L2VPN 0000.0001.0101   172.16.101.10                  Et0/0:101
    101          0   BGP 0000.0002.0101   172.16.101.11         V:10101 10.1.254.4
    101          0   BGP 0000.0003.0101   172.16.101.12         V:10101 10.1.254.5
    101          0   BGP aabb.cc80.0400    172.16.101.1         V:10101 10.1.254.4
    101          0   BGP aabb.cc80.0500    172.16.101.1         V:10101 10.1.254.5
    101          0   BGP aabb.cc80.0600    172.16.101.1         V:10101 10.1.254.6
    101          0   BGP aabb.cc80.0700    172.16.101.1         V:10101 10.1.254.7
    <...skip...>

Do those 2 VNIs exist on the switch? Looks like ``10110`` does not exist – in the configuration of NVE we can find out which VNI is actually expected to be here.

.. code-block:: console
    :linenos:
    :emphasize-lines: 3,7,14,21
    :class: highlight-command emphasize-hll

    ts01-L1#sh nve vni 10101
    Interface  VNI        Multicast-group VNI state  Mode  VLAN  cfg vrf                      
    nve1       10101      N/A             BD Down/Re L2CP  N/A   CLI N/A    

    ts01-L1#sh nve vni 10110 detail 
    Interface  VNI        Multicast-group VNI state  Mode  VLAN  cfg vrf                      
    % VNI 10110 doesn't exist

    ts01-L1#sh run int nve1
    interface nve1
     no ip address
     source-interface Loopback1
     host-reachability protocol bgp
     member vni 10101 ingress-replication 
     member vni 10102 mcast-group 225.0.1.102
     member vni 50901 vrf green
     end

    ts01-L1#sh run vlan 101
    vlan configuration 101
     member evpn-instance 101 vni 10110 

We have identified that there is a mismatch in vlan-to-VNI mapping, as for vlan 101 L2VNI ``10110`` is used instead of the expected VNI ``10101``. Correct L2VNI is not configured on the switch.

Lets fix the configuration mistake on L1 node and reconfigure the NVE-VNI membership to retrigger the NVE peer learning for VNI ``10101``.

L1 node

.. code-block:: console
    :linenos:

    conf t
    no vlan configuration 101
    vlan configuration 101
     member evpn-instance 101 vni 10101
    !
    int nve1
     no member vni 10101 ingress-replication
     member vni 10101 ingress-replication

Checking the NVE peers for that VNI afterwards, we see remote Leafs and connectivity start working.

L1 node

.. code-block:: console
    :linenos:
    :class: highlight-command

    ts01-L1#sh nve peers  vni 10101
    'M' - MAC entry download flag  'A' - Adjacency download flag
    '4' - IPv4 flag  '6' - IPv6 flag

    Interface  VNI      Type Peer-IP          RMAC/Num_RTs   eVNI     state flags UP time
    nve1       10101    L2CP 10.1.254.4       5              10101      UP   N/A  00:00:23
    nve1       10101    L2CP 10.1.254.5       5              10101      UP   N/A  00:00:23
    nve1       10101    L2CP 10.1.254.6       3              10101      UP   N/A  00:00:23
    nve1       10101    L2CP 10.1.254.7       3              10101      UP   N/A  00:00:23

H1 node

.. code-block:: console
    :linenos:
    :class: highlight-command

    ts01-H1#ping vrf h1 172.16.101.11          
    Type escape sequence to abort.
    Sending 5, 100-byte ICMP Echos to 172.16.101.11, timeout is 2 seconds:
    !!!!!
    Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/1 ms
