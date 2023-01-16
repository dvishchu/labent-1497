Task CFG01: Configure L2/L3 + Spines EVPN connectivity with Spines
==================================================================

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

    stitching is a new keyword added to the existing route-target configuration to specify the route targets to be used when doing EVPN related processing.

    EVPN->VRF
        EVPN routes that contain RT matching an import “stitching RT” specified in a VRF configuration are accepted by the router and imported into the corresponding BGP L3VPN VRF. The resulting L3VPN prefix retains the same route target. 

    VRF->EVPN
        L3VPN routes that are imported into EVPN via “advertise l2vpn evpn” contain RTs specified by that VRF export “stitching RT”. Any original route targets are removed.

    The existing RT configuration does not affect EVPN related processing, and you can have the same RT values for both base and VXLAN EVPN routes. 

VRF name is ``green``. RT ``1:1`` is to be used for base route target configuration and RT ``10:10`` for EVPN related processing.

L1/L2/L3 nodes

.. code-block:: console

    conf t
    vrf def green
     rd 1:1
     address-family ipv4 unicast
      route-target both 1:1
      route-target both 10:10 stitching
      end
