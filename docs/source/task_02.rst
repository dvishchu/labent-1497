Task CFG02: Configure Border Leafs and L3 routing between external network and Fabric
=====================================================================================

.. image:: assets/cfg02_topology.png
    :align: center

In this task we are configuring the more complex topology with a connectivity between fabric and external networks, using the Border Leaf switches.

.. note::

    External connectivity allows the movement of Layer 2 and Layer 3 traffic between an EVPN VXLAN network and an external network. It also enables the EVPN VXLAN network to exchange routes with the externally connected network. 

    Routes within an EVPN VXLAN network are already shared between all the VTEPs/Leafs. 

    External connectivity uses the Leafs on the periphery of the network to pass on these routes to an external Layer 2 or Layer 3 network. Similarly, the EVPN VXLAN network imports the reachability routes from the external network.

To get started, please select in ``lab manager`` option ``02`` to initialize lab devices.

.. note::

    At the beginning of the task Border Leafs are fully integrated to the fabric, External network is configured.

    L3 interfaces in a dedicated VRF “green” are used on Border Leafs for external connectivity between borders and external nodes.


Step 1: Add IP interfaces to BL1/2 and EXT1/2
*********************************************

.. image:: assets/cfg02_step1_topology.png
    :align: center

First, we need to configure underlay and OSPF for routes exchange (area 0 is used for the lab scenario). Note that Border Leaf 1 and 2 interfaces towards EXT nodes are part of VRF “green”.

EXT1 node

.. code-block:: console

    conf t
    !
    interface e1/1
     no sw
     no shut
     ip addr 192.168.68.8 255.255.255.0
     ip ospf 100 area 0
     ip ospf net point-to-point
    !
    interface e1/2
     no sw
     no shut
     ip addr 192.168.78.8 255.255.255.0
     ip ospf 100 area 0
     ip ospf net point-to-point

EXT2 node

.. code-block:: console

    conf t
    !
    interface e1/1
     no sw
     no shut
     ip addr 192.168.69.9 255.255.255.0
     ip ospf 100 area 0
     ip ospf net point-to-point
    !
    interface e1/2
     no sw
     no shut
     ip addr 192.168.79.9 255.255.255.0
     ip ospf 100 area 0
     ip ospf net point-to-point

BL1 node

.. code-block:: console

    conf t
    !
    router ospf 100 vrf green
     router-id 172.16.255.6
    !
    interface e1/1
     no sw
     no shut
     vrf for green
     ip addr 192.168.68.6 255.255.255.0
     ip ospf 100 area 0
     ip ospf net point-to-point
    !
    interface e1/2
     no sw
     no shut
     vrf for green
     ip addr 192.168.69.6 255.255.255.0
     ip ospf 100 area 0
     ip ospf net point-to-point

BL2 node

.. code-block:: console

    conf t
    !
    router ospf 100 vrf green
     router-id 172.16.255.7
    !
    interface e1/1
     no sw
     no shut
     vrf for green
     ip addr 192.168.78.7 255.255.255.0
     ip ospf 100 area 0
     ip ospf net point-to-point
    !
    interface e1/2
     no sw
     no shut
     vrf for green
     ip addr 192.168.79.7 255.255.255.0
     ip ospf 100 area 0
     ip ospf net point-to-point

To verify that OSPF is converged properly, check the neighborship status and routes exchange:

BL1 node

.. code-block:: console

    cfg04-BL1#sh ip ospf 100 nei
    Neighbor ID     Pri   State           Dead Time   Address         Interface
    192.168.255.9     0   FULL/  -        00:00:30    192.168.69.9    Ethernet1/2
    192.168.255.8     0   FULL/  -        00:00:35    192.168.68.8    Ethernet1/1

    cfg04-BL1#sh ip ro vrf green ospf | b Gateway

    O     192.168.78.0/24 [110/20] via 192.168.68.8, 00:10:52, Ethernet1/1
    O     192.168.79.0/24 [110/20] via 192.168.69.9, 00:10:49, Ethernet1/2
    O     192.168.89.0/24 [110/20] via 192.168.69.9, 00:10:49, Ethernet1/2
                        [110/20] via 192.168.68.8, 00:10:52, Ethernet1/1
    O IA  192.168.201.0/24 [110/11] via 192.168.68.8, 00:10:52, Ethernet1/1
        192.168.255.0/32 is subnetted, 2 subnets
    O        192.168.255.8 [110/11] via 192.168.68.8, 00:10:52, Ethernet1/1
    O        192.168.255.9 [110/11] via 192.168.69.9, 00:10:49, Ethernet1/2

BL2 node 

.. code-block:: console

    cfg04-BL2#sh ip ospf 100 nei
    Neighbor ID     Pri   State           Dead Time   Address         Interface
    192.168.255.9     0   FULL/  -        00:00:34    192.168.79.9    Ethernet1/2
    192.168.255.8     0   FULL/  -        00:00:31    192.168.78.8    Ethernet1/1

    cfg04-BL2#sh ip ro vrf green ospf | b Gateway
    O     192.168.68.0/24 [110/20] via 192.168.78.8, 00:10:57, Ethernet1/1
    O     192.168.69.0/24 [110/20] via 192.168.79.9, 00:10:55, Ethernet1/2
    O     192.168.89.0/24 [110/20] via 192.168.79.9, 00:10:55, Ethernet1/2
                        [110/20] via 192.168.78.8, 00:10:57, Ethernet1/1
    O IA  192.168.201.0/24 [110/11] via 192.168.78.8, 00:10:57, Ethernet1/1
        192.168.255.0/32 is subnetted, 2 subnets
    O        192.168.255.8 [110/11] via 192.168.78.8, 00:10:57, Ethernet1/1
    O        192.168.255.9 [110/11] via 192.168.79.9, 00:10:55, Ethernet1/2

EXT1 node

.. code-block:: console

    cfg04-EXT1#sh ip ospf nei
    Neighbor ID     Pri   State           Dead Time   Address         Interface
    172.16.255.7      0   FULL/  -        00:00:32    192.168.78.7    Ethernet1/2
    172.16.255.6      0   FULL/  -        00:00:33    192.168.68.6    Ethernet1/1
    192.168.255.9     0   FULL/  -        00:00:34    192.168.89.9    Ethernet0/3

EXT2 node

.. code-block:: console

    cfg04-EXT2#sh ip ospf nei
    Neighbor ID     Pri   State           Dead Time   Address         Interface
    172.16.255.7      0   FULL/  -        00:00:34    192.168.79.7    Ethernet1/2
    172.16.255.6      0   FULL/  -        00:00:32    192.168.69.6    Ethernet1/1
    192.168.255.8     0   FULL/  -        00:00:39    192.168.89.8    Ethernet0/3


Step 2: Redistribute OSPF 100 to BGP 65001 and vice versa on BL1/2
******************************************************************