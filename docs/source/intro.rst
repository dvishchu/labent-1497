Get started
===============

Cisco Anyconnect + RDP Option
*****************************

#. Please connect to the lab with **Cisco AnyConnect** (you can get server name and credentials from the lab proctor):

    #. Start **Cisco AnyConnect** on your laptop
    #. Copy the **Host** URL from the **AnyConnect Credentials** and then paste it in the **URL Connection box** in the **AnyConnect** login window.
    #. Click connect

        .. image:: assets/anyconnect.png

    #. If you get a connection error, remove the ``https://`` part of the URL and try the connection again.
    #. Copy a User ID and the password from the **AnyConnect Credentials** and then paste each into the **Cisco AnyConnect** login window.
    #. Click **OK**
    #. Click **Accept** on the window confirming your connection.
    #. When connected to your AnyConnect VPN session, the AnyConnect VPN icon is displayed in the system tray (Windows) or task bar (Mac).
    #. To view connection details or to disconnect, click the AnyConnect VPN icon and then choose Disconnect.

#. After connecting to the VPN of the lab pod, use RDP to connect to the jumphost workstation.

    **IP Address**: 198.18.133.36

    **Credentials**: admin / C1sco12345

WebRDP Option
*************

Alternatively to using Cisco Anyconnect and RDP applications, you can connect to the lab environment using WebRDP access to the jumphost workstation via the browser.

.. note:: 

    This connectivity option may limit the functionality to copy and paste from your laptop to the lab device consoles

Open the the WebRDP link provided for your lab session by proctor, using your CCO credentials to login into dCloud environment.