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
开启Leap容灾之旅
-------------------------

概述
++++++++

围绕本地故障转移操作的 AOS 5.17 引入了对 Leap 的重大改进，包括支持Guest脚本的执行以及与 AHV 的同步复制。

在这个实验中你将会:

- 执行多层应用 - Fiesta - 由Calm部署
- 使用保护策略保护您的虚拟机
- 为 Runbook 自动化制定恢复计划
- 创建对Fiesta应用程序数据库的更改
- 对物理 Nutanix 集群执行故障转移
- 访问Fiesta应用程序以确保保留更改
- 对Fiesta应用程序数据库进行其他更改
- 从物理 Nutanix 集群执行故障恢复
- 访问Fiesta应用程序以确保保留更改

这个实验为 **Unplanned** 和 **Planned** 故障转移场景提供练习，完成一个实验后，您可以跳过第二个实验的配置步骤。 您将在这两个实验中使用相同的虚拟机、保护策略和恢复计划。 这在每个实验的介绍中也有详细说明。

更新
++++++++++

- 实验操作使用以下软件版本更新:

   - AOS - 5.19.2
   - AHV - el7.nutanix.20201105.1161
   - Prism Central - pc.2021.3.0.1

环境要求
++++++++++++++++++++++++

- 两个运行 AOS 5.17.1/AHV 20190916.189 或更新版本的 4 节点 AHV 集群。 在同一个 HPOC 数据中心预留两个 4 节点集群，以确保满足同步复制延迟要求（<5 毫秒 RTT）。

   .. note::

      Leap 的恢复计划要求两个集群具有相同的网络前缀长度（例如 /25、/26）。 此外，由于需要 4 节点集群。 不要在此实验环境中混合使用单节点和 4 节点集群。


- 按照以下步骤打开所需的端口: 2030, 2036, 2073, and 2090.

   .. note::

      即使在同一数据中心使用两个 HPOC 集群，此步骤也是必要的。

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
