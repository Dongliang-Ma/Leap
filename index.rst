.. title:: Leap Add-On Bootcamp

.. toctree::
   :maxdepth: 2
   :name: _onpremleap
   :caption: Leap Labs
   :hidden:

   onpremleap3_PFO/onpremleap3_PFO
   onpremleap2_UPFO/onpremleap2_UPFO

.. .. toctree::
..    :maxdepth: 2
..    :name: _onpremleap
..    :caption: Single PC (TESTING)
..    :hidden:
..
..    1PC/onpremleap3_PFO_1pc/onpremleap3_PFO_1pc
..    1PC/onpremleap2_UPFO_1pc/onpremleap2_UPFO_1pc

-------------------------
Getting Started with Leap
-------------------------

Overview
++++++++

Significant enhancements to Leap were introduced with AOS 5.17 surrounding on-premises failover operations, including support for execution of guest scripts, and synchronous replication with AHV.

In this lab you will:

- Use a multi-tier application - Fiesta - deployed by Calm
- Protect your VMs with a Protection Policy
- Build a Recovery Plan for runbook automation
- Create changes to the Fiesta application database
- Perform a failover to a physical Nutanix cluster
- Access the Fiesta application to ensure changes were retained
- Make additional changes to the Fiesta application database
- Perform a failback from a physical Nutanix cluster
- Access the Fiesta application to ensure changes were retained

This add-on Bootcamp provides exercises for BOTH **Unplanned** and **Planned** failover scenarios. As you complete one exercise, you may skip the configuration steps for the second exercise. You will use the same VMs, Protection Policy, and Recovery Plan for both exercises. This is also detailed in the introduction to each lab.

What's New
++++++++++

- Workshop updated using the following software versions:

   - AOS - 5.19.2
   - AHV - el7.nutanix.20201105.1161
   - Prism Central - pc.2021.3.0.1

Environment Requirements
++++++++++++++++++++++++

- Two 4-node AHV clusters running AOS 5.17.1/AHV 20190916.189 or newer. Reserve two 4-node clusters in the same HPOC datacenter to ensure synchronous replication latency requirements are met (<5ms RTT).

   .. note::

      Leap's Recovery Plans require both clusters to have the same network prefix length (ex. /25, /26). Also, as this is meant as an add-on bootcamp, 4-node clusters are required. Do not mix single-node and 4-node clusters in this bootcamp environment.

- You can run any standard Bootcamp staging on one cluster (e.g. **Enterprise Private Cloud Bootcamp**, **Databases: Era with MSSQL Bootcamp**, etc.) This will be your *Recovery* site cluster.

- You **must** run the **Leap Add-On Bootcamp** staging on your additional cluster. This will be your *Primary* site cluster.

- Follow the steps below to open the required ports: 2030, 2036, 2073, and 2090.

   .. note::

      This step is necessary even if using two HPOC clusters in the same datacenter.

   #. To open the ports for communication to the recovery cluster, run the following command on a CVM for the *Primary* site cluster by remoting in via SSH (e.g. ssh nutanix@<CLUSTER-VIRTUAL-IP>):

      .. code-block:: bash

          allssh 'modify_firewall -f -r recovery_cvm_ip,recovery_virtual_ip -p 2030,2036,2073,2090 -i eth0'

      Replace remote_cvm_ip with the IP addresses of the primary cluster CVMs, separated by a comma.

      Replace remote_virtual_ip with the virtual IP address of the recovery cluster.

   #. To open the ports for communication to the primary cluster, run the following command on a CVM for the *Recovery* site cluster by remoting in via SSH (e.g. ssh nutanix@<CLUSTER-VIRTUAL-IP>):

      .. code-block:: bash

         allssh 'modify_firewall -f -r primary_cvm_ip,primary_virtual_ip -p 2030,2036,2073,2090 -i eth0'

      Replace `source_cvm_ip` with the IP addresses of the primary cluster CVMs, separated by a comma.

      Replace `source_virtual_ip` with the virtual IP address of the primary cluster.

   #. Exit both SSH sessions.

   .. note::

      For more information about the required ports, see the **General Requirements of Leap** section within the `Xi Leap Administration Guide <https://portal.nutanix.com/page/documents/details?targetId=Leap-Xi-Leap-Admin-Guide-v5_19:Leap-Xi-Leap-Admin-Guide-v5_19>`_.

- This staging script will automatically deploy **10** instances of the Fiesta application:

   - **User01-MYSQL-...**; **User01-WebServer-...**
   - **User02-MYSQL-...**; **User02-WebServer-...**
   - And so on...

- The instructor should assign a *User* number to each participant. The lab guide will reference entity names with *UserXX* which should be substituted for their specific number (e.g. *User01*).

- It is recommended to rename the clusters within Prism to *Primary* and *Recovery* respectively. This will aid in identification during the labs.

Reference
+++++++++

The following resources are provided for reference purposes, and aren't required to complete the lab exercises.

- `Xi Leap Administration Guide <https://portal.nutanix.com/page/documents/details?targetId=Leap-Xi-Leap-Admin-Guide-v5_19:Leap-Xi-Leap-Admin-Guide-v5_19>`_
