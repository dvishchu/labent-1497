Task CFG01: Configure L2/L3 + Spines EVPN connectivity with Spines
==================================================================

.. image:: assets/cfg01_topology.png
    :align: center

In this task, we will be exploring the example of the Leafs+Spines topology with the addition of L3 VNI.

An EVPN VXLAN Layer 3 overlay network allows host devices in different Layer 2 networks to send Layer 3 or routed traffic to each other. The network forwards the routed traffic using a Layer 3 virtual network instance (VNI) and an IP VRF.

To get started, please select in ``lab manager`` option ``01`` to initialize lab devices.

.. note::

    At the beginning of the task the following protocols are pre-configured and running:
        * IGP is UP
        * BGP is up
        * Multicast is configured
        * L2VNI, EVI, VNI are configured and up


Step 1: Create VRF
******************

First step to configure L3 VNI routing is to have VRF defined with RD (route distinguisher) and import/export RTs (route targets) correctly configured.

.. note::

    ``stitching`` is a new keyword added to the existing route-target configuration to specify the route targets to be used when doing EVPN related processing.

    EVPN->VRF
        EVPN routes that contain RT matching an import “stitching RT” specified in a VRF configuration are accepted by the router and imported into the corresponding BGP L3VPN VRF. The resulting L3VPN prefix retains the same route target. 

    VRF->EVPN
        L3VPN routes that are imported into EVPN via “advertise l2vpn evpn” contain RTs specified by that VRF export “stitching RT”. Any original route targets are removed.

    The existing RT configuration does not affect EVPN related processing, and you can have the same RT values for both base and VXLAN EVPN routes. 

VRF name is ``green``. RT ``1:1`` is to be used for base route target configuration and RT ``10:10`` for EVPN related processing.

L1/L2/L3 nodes

.. code-block:: console

    conf t
    !
    vrf def green
     rd 1:1
     address-family ipv4 unicast
      route-target both 1:1
      route-target both 10:10 stitching

You can check results with the ``show vrf detail <VRF_Name>`` command, e.g.:

L1 node

.. code-block:: console

    cfg03-L1#sh vrf detail green
    VRF green (VRF Id = 1); default RD 1:1; default VPNID <not set>
    New CLI format, supports multiple address-families
    Flags: 0x180C
    No interfaces
    Address family ipv4 unicast (Table ID = 0x1):
    Flags: 0x0
    Export VPN route-target communities
        RT:1:1
    Import VPN route-target communities
        RT:1:1
    Export VPN route-target stitching communities
        RT:10:10
    Import VPN route-target stitching communities
        RT:10:10
    No import route-map
    No global export route-map
    No export route-map


Step 2: Configure MAC Aliasing for the distributed anycast gateway
******************************************************************

.. note::

    Distributed anycast gateway is a default gateway addressing mechanism in a BGP EVPN VXLAN fabric.

    This feature enables the use of the same gateway IP and MAC address across all the Leafs in an EVPN VXLAN network, to ensure that every Leaf functions as the default gateway for the workloads directly connected to it. The feature facilitates flexible workload placement, host mobility, and optimal traffic forwarding across the BGP EVPN VXLAN fabric. 

In our lab scenario we are using ``MAC aliasing``, which allows the Leafs to advertise their VLAN MAC addresses as the gateway MAC addresses to all the other Leafs in the network. The Leafs in the network store the advertised MAC address as a gateway MAC address provided their VLAN IP address matches with the gateway IP address.

Alternative way (not shown in the lab scenarios) would be to manually configure the same MAC address on the VLAN interfaces of all Leaf switches in the network. 

L1/L2/L3 nodes

.. code-block:: console

    conf t
    !
    l2vpn evpn
    default-gateway advertise 

Verification output is part of the ``sh l2vpn evpn summary`` command:

.. code-block:: console

      cfg03-L1#sh l2vpn evpn summary | i Default
      Advertise Default Gateway: Yes
      Default Gateway Addresses: 0

      cfg03-L2#sh l2vpn evpn summary | i Default
      Advertise Default Gateway: Yes
      Default Gateway Addresses: 0

      cfg03-L3#sh l2vpn evpn summary | i Default
      Advertise Default Gateway: Yes
      Default Gateway Addresses: 0

Step 3: Create VNI to vlan stitching for vlan901 (L3VNI), create SVIs for L2VNIs and L3VNI
******************************************************************************************

At this step, we create vlan 901 and SVI 901 to be mapped to L3VNI 50901. Similarly, we create SVIs for L2VNIs for routing between L2 domains. 

    * All SVI interfaces are part of “green” VRF. 
    * For L3VNI SVI make sure to enable IP processing on the Loopback1 interface without assigning an explicit IP address to the SVI.

.. list-table::
    :widths: 33 33 33
    :header-rows: 1
    :width: 100%

    * - VLAN
      - VNI
      - IP Address
    * - 101
      - 102
      - 901
    * - 10101
      - 10102
      - 50901
    * - 172.16.101.1
      - 172.16.102.1
      - ip unnumbered lo0

.. image:: assets/cfg01_vni.png
    :align: center

L1/L2/L3 nodes

.. code-block:: console

    conf t
    !
    vlan 901
    !
    vlan configuration 901
    member vni 50901
    !
    interface Vlan101
    vrf forwarding green
    ip address 172.16.101.1 255.255.255.0
    no shut
    !
    interface Vlan102
    vrf forwarding green
    ip address 172.16.102.1 255.255.255.0
    no shut
    !
    interface vlan901
    vrf forwarding green
    ip unnumbered lo1
    no autostate
    no shut

.. note::

    Same gateway IP and MAC address are used for L2VNI SVI interfaces across all the Leafs, to make a distributed anycast gateway.

