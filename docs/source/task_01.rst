Task CFG01: Configure L2 EVPN connectivity with Spines
======================================================

In this task, we will be configuring L2 connectivity in EVPN fabric. L2 connectivity will be extended across the leaf switches which will allow hosts connected to different leafs communicate with each other as they would be connected to same leaf. To achieve this, we will configure L2 VNI and hosts will be able to communicate within same L2 domain only.  

To get started, please select in ``lab manager`` option ``01`` to initialize lab devices.

.. note::

    At beginning of this task, underlay configuration was already preconfigured for you. This includes configuration of IGP - OSPF for loopback reachability and multicast routing to support flooding of broadcast  / multicast / unknown unicast (BUM) traffic.

Step 1: Configure L2VNP and VNI-EVI-VLAN stitching
**************************************************

We will start with L2VPN configuration. Configuration can be defined either in global context (l2vpn evpn) or per instance (l2vpn evpn instance …). In case that settings are not overridden in per instance context, they are inherited from global. 

First, we will define replication type based on which we will be flooding BUM traffic in the fabric. The BGP EVPN control plane uses two methods for this purpose: multicast based (static), where multicast is flooded over dedicated multicast group to rest of the leaf switches or unicast based (ingress replication - ingress), where BUM traffic is delivered to leaf switched over unicast. 

In this scenario, we will configure global context with replication type ingress (unicast based) and define two EVI isntances:
    
    * EVI 101 – replication type ingress (unicast based), VNI 10101
    * EVI 102 – replication type static (multicast based), VNI 10102

.. note::

    VXLAN encapsulation format is a default setting. 

    EVPN instance needs to be explicitly configured only when something needs to be configured per EVPN instance such as a route target, encapsulation or replication type.

L1/L2/L3 nodes

.. code-block:: console
    :linenos:

    conf t
    !
    l2vpn evpn
     replication-type ingress
    !
    l2vpn evpn instance 101 vlan-based
     encapsulation vxlan
    !
    l2vpn evpn instance 102 vlan-based
     encapsulation vxlan
     replication-type static
    !
    vlan configuration 101
     member evpn-instance 101 vni 10101
    vlan configuration 102
     member evpn-instance 102 vni 10102


Step 2: Configure NVE interface
*******************************

Next, we have to configure network virtualization endpoint (NVE) interface. The NVE interface is logical interface where encapsulation and decapsulation of traffic happens for VXLAN traffic.  

Specified replication type defined on NVE interface have to match replication type on EVI instance. In case of multicast based replication (static), we have to define multicast group which will be used for flooding of BUM traffic. In case of unicast based replication (ingress), this is not needed since BGP control plane will built list of leafs where BUM traffic have to be replicated via unicast. 

L1/L2/L3 nodes

.. code-block:: console
    :linenos:

    conf t
    !
    interface nve1
     no ip address
     source-interface Loopback1
     host-reachability protocol bgp
     member vni 10101 ingress-replication
     member vni 10102 mcast-group 225.0.1.102


Step 3: Configure BGP
*********************

As last step, we will configure BGP protocol, so we can advertise host reachability information via L2VPN EVPN address family in fabric. In this scenario, both spine and leaf switches are part of same AS 65001 and spine switches are acting like route reflectors.

L1/L2/L3 node

.. code-block:: console
    :linenos:

    conf t
    !
    router bgp 65001
     no bgp default ipv4-unicast
     neighbor 10.1.255.1 remote-as 65001
     neighbor 10.1.255.1 update-source Loopback0
     neighbor 10.1.255.2 remote-as 65001
     neighbor 10.1.255.2 update-source Loopback0
     !
     address-family l2vpn evpn
      neighbor 10.1.255.1 activate
      neighbor 10.1.255.1 send-community both
      neighbor 10.1.255.2 activate
      neighbor 10.1.255.2 send-community both

S1/S2 node

.. code-block:: console
    :linenos:

    conf t
    !
    router bgp 65001
     no bgp default ipv4-unicast
     neighbor 10.1.255.3 remote-as 65001
     neighbor 10.1.255.3 update-source Loopback0
     neighbor 10.1.255.4 remote-as 65001
     neighbor 10.1.255.4 update-source Loopback0
     neighbor 10.1.255.5 remote-as 65001
     neighbor 10.1.255.5 update-source Loopback0
     !
     address-family l2vpn evpn
      neighbor 10.1.255.3 activate
      neighbor 10.1.255.3 send-community both
      neighbor 10.1.255.3 route-reflector-client
      neighbor 10.1.255.4 activate
      neighbor 10.1.255.4 send-community both
      neighbor 10.1.255.4 route-reflector-client
      neighbor 10.1.255.5 activate
      neighbor 10.1.255.5 send-community both
      neighbor 10.1.255.5 route-reflector-client


Step 4: Verification
********************