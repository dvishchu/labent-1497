Task TS02: H1(172.16.101.10) cannot ping H3(172.16.101.12) over vxlan via vlan101
=================================================================================

.. image:: assets/cfg01_topology.png
    :align: center

H1 node 

.. code-block:: console
    :linenos:
    :emphasize-lines: 4,10
    :class: highlight-command

    ts02-H1#ping vrf h1 172.16.101.12
    Type escape sequence to abort.
    Sending 5, 100-byte ICMP Echos to 172.16.101.12, timeout is 2 seconds:
    .....
    Success rate is 0 percent (0/5)

    ts02-H1#ping vrf h1 172.16.101.1 
    Type escape sequence to abort.
    Sending 5, 100-byte ICMP Echos to 172.16.101.1, timeout is 2 seconds:
    !!!!!
    Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/1 ms

We can see that NVE peering looks fine on Leaf1.

L1 node 

.. code-block:: console
    :linenos:
    :class: highlight-command

    ts02-L1#sh nve peer    
    'M' - MAC entry download flag  'A' - Adjacency download flag
    '4' - IPv4 flag  '6' - IPv6 flag

    Interface  VNI      Type Peer-IP          RMAC/Num_RTs   eVNI     state flags UP time
    nve1       50901    L3CP 10.1.254.4       aabb.cc80.0400 50901      UP  A/M/4 00:00:03
    nve1       50901    L3CP 10.1.254.6       aabb.cc80.0600 50901      UP  A/M/4 00:00:03
    nve1       50901    L3CP 10.1.254.7       aabb.cc80.0700 50901      UP  A/M/4 00:00:03
    nve1       10101    L2CP 10.1.254.4       5              10101      UP   N/A  00:00:03
    nve1       10101    L2CP 10.1.254.6       3              10101      UP   N/A  00:00:03
    nve1       10101    L2CP 10.1.254.7       3              10101      UP   N/A  00:00:03
    nve1       10102    L2CP 10.1.254.4       4              10102      UP   N/A  00:00:03
    nve1       10102    L2CP 10.1.254.6       2              10102      UP   N/A  00:00:03
    nve1       10102    L2CP 10.1.254.7       2              10102      UP   N/A  00:00:03

BGP is up as well and receives prefixes from neighbors.

.. code-block:: console
    :linenos:
    :class: highlight-command

    ts02-L1#sh bgp l2vpn evpn summary 
    BGP router identifier 10.1.255.3, local AS number 65001

    Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
    10.1.255.1      4        65001      23      10       44    0    0 00:01:31       23
    10.1.255.2      4        65001      23       9       44    0    0 00:01:26       23

    ts02-L1#sh l2route evpn mac ip | i 101 
    101          0 L2VPN 0000.0001.0101   172.16.101.10                  Et0/0:101
    101          0   BGP 0000.0002.0101   172.16.101.11         V:10101 10.1.254.4
    101          0 L2VPN aabb.cc80.0300    172.16.101.1                    Vl101:0
    101          0   BGP aabb.cc80.0400    172.16.101.1         V:10101 10.1.254.4
    101          0   BGP aabb.cc80.0600    172.16.101.1         V:10101 10.1.254.6
    101          0   BGP aabb.cc80.0700    172.16.101.1         V:10101 10.1.254.7

We see, however, that 172.16.101.12 prefix is not present in the list of the EVPN MAC/IP info. 

Lets check if we have this route in BGP table – the output below confirms that such route is not present, MAC for the destination host is 0000.0003.0101.

.. code-block:: console
    :linenos:
    :class: highlight-command

    ts02-L1#sh bgp l2vpn evpn route-type 2 0 000000030101 172.16.101.12
    % Network not in table 

Is it present on RRs (spines)?

S1 node 

.. code-block:: console
    :linenos:
    :class: highlight-command

    ts02-S1#sh bgp l2vpn evpn route-type 2 0 000000030101 172.16.101.12
    BGP routing table entry for [2][10.1.255.5:101][0][48][000000030101][32][172.16.101.12]/24, version 61
    Paths: (2 available, best #1, table EVPN-BGP-Table)
    Advertised to update-groups:
        2
    Refresh Epoch 1
    Local
        10.1.254.5 (metric 11) (via default) from 10.1.255.5 (10.1.255.5)
        Origin incomplete, metric 0, localpref 100, valid, internal, best
        EVPN ESI: 00000000000000000000, Label1 10101, Label2 50901
        Extended Community: RT:10:10 RT:65001:101 ENCAP:8
            Router MAC:AABB.CC80.0500
        rx pathid: 0, tx pathid: 0x0
        Updated on Mar 25 2021 20:06:24 CET
    Refresh Epoch 1
    Local, (Received from a RR-client)
        10.1.254.5 (metric 11) (via default) from 10.1.255.2 (10.1.255.2)
        Origin incomplete, metric 0, localpref 100, valid, internal
        EVPN ESI: 00000000000000000000, Label1 10101, Label2 50901
        Extended Community: RT:10:10 RT:65001:101 ENCAP:8
            Router MAC:AABB.CC80.0500
        Originator: 10.1.255.5, Cluster list: 10.1.255.2
        rx pathid: 0, tx pathid: 0
        Updated on Mar 25 2021 20:06:24 CET

.. note::

    The update-group might be different in your lab!

Route is present and is being advertised to the BGP update-group (note the group number in the output above). Lets see which routers are part of it.

.. code-block:: console
    :linenos:
    :class: highlight-command

    ts02-S1#sh bgp l2vpn evpn update-group 2 
    BGP version 4 update-group 2, internal, Address Family: L2VPN E-VPN
    BGP Update version : 79/0, messages 0, active RGs: 1
    Route-Reflector Client
    Community attribute sent to this neighbor
    Extended-community attribute sent to this neighbor
    Topology: global, highest version: 79, tail marker: 79
    Format state: Current working (OK, last not in list)
        Refresh blocked (not in list, last not in list)
    Update messages formatted 67, replicated 268, current 0, refresh 0, limit 1000, mss 1460, SSO is disabled
    Number of NLRIs in the update sent: max 2, min 0
    Minimum time between advertisement runs is 0 seconds
    Has 4 members:
        10.1.255.2       10.1.255.4       10.1.255.6       10.1.255.7      

Looking into the update-group members, peer ``10.1.255.3`` is not part of it. 

To identify the reason for this issue, we will check the BGP config for problem and working neighbors.

.. code-block:: console
    :linenos:
    :class: highlight-command

    ts02-S1#sh bgp l2vpn evpn neighbors 10.1.255.3 | b L2VPN E-VPN
    For address family: L2VPN E-VPN
    Session: 10.1.255.3
    BGP table version 62, neighbor version 62/0
    Output queue size : 0
    Index 2, Advertise bit 1
    2 update-group member
    Community attribute sent to this neighbor
    Extended-community attribute sent to this neighbor
    Slow-peer detection is disabled
    Slow-peer split-update-group dynamic is disabled
    Prefers VxLAN if VTEP is UP else MPLS 
                                    Sent       Rcvd
    Prefix activity:               ----       ----
        Prefixes Current:              21          5 (Consumes 1120 bytes)
        Prefixes Total:                37          9
    <...skip...>

    ts02-S1#sh bgp l2vpn evpn neighbors 10.1.255.4 | b L2VPN E-VPN
    For address family: L2VPN E-VPN
    Session: 10.1.255.4
    BGP table version 62, neighbor version 62/0
    Output queue size : 0
    Index 1, Advertise bit 0
    Route-Reflector Client 
    1 update-group member
    Community attribute sent to this neighbor
    Extended-community attribute sent to this neighbor
    Slow-peer detection is disabled
    Slow-peer split-update-group dynamic is disabled
    Prefers VxLAN if VTEP is UP else MPLS 
                                    Sent       Rcvd
    Prefix activity:               ----       ----
        Prefixes Current:              31          5 (Consumes 1120 bytes)
    <...skip....>

Looks like route-reflector command is missing for the 10.1.255.3 neighbor. That configuration command is required since S1 node is acting as a Spine in the EVPN fabric.

Similarly, such configuration is missing on S2 node too. 

S2 node

.. code-block:: console
    :linenos:
    :class: highlight-command

    ts02-S2#sh bgp l2vpn evpn neighbors 10.1.255.3 | b L2VPN E-VPN
    For address family: L2VPN E-VPN
    Session: 10.1.255.3
    BGP table version 101, neighbor version 101/0
    Output queue size : 0
    Index 1, Advertise bit 0
    1 update-group member
    Community attribute sent to this neighbor
    Extended-community attribute sent to this neighbor
    Slow-peer detection is disabled
    Slow-peer split-update-group dynamic is disabled
    Prefers VxLAN if VTEP is UP else MPLS
                                    Sent       Rcvd
    Prefix activity:               ----       ----
        Prefixes Current:              21          5 (Consumes 1120 bytes)
        Prefixes Total:                54         13
        Implicit Withdraw:             14          0
        Explicit Withdraw:             19          8
        Used as bestpath:             n/a          5
        Used as multipath:            n/a          0
        Used as secondary:            n/a          0

Lets fix it on S1 and S2 nodes (make sure to do it on both Spines).

S1/S2 nodes

.. code-block:: console
    :linenos:
    :class: highlight-command

    conf t
     router bgp 65001
      address-family l2vpn evpn
       neighbor 10.1.255.3 route-reflector-client

After that we will see 172.16.101.12 in l2route table of Leaf1.

.. code-block:: console
    :linenos:
    :class: highlight-command

    ts02-L1#sh l2route evpn mac ip | i 101                             
    101          0 L2VPN 0000.0001.0101   172.16.101.10                  Et0/0:101
    101          0   BGP 0000.0002.0101   172.16.101.11         V:10101 10.1.254.4
    101          0   BGP 0000.0003.0101   172.16.101.12         V:10101 10.1.254.5 
    <...skip....>

Lets try to ping from H1 to verify.

.. code-block:: console
    :linenos:
    :class: highlight-command

    ts02-H1#ping vrf h1 172.16.101.12
    Type escape sequence to abort.
    Sending 5, 100-byte ICMP Echos to 172.16.101.12, timeout is 2 seconds:
    .!!!!
