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

   #. 要打开与恢复集群通信的端口，请通过 SSH（例如 ssh nutanix@<CLUSTER-VIRTUAL-IP>）在 *Primary* 站点集群的 CVM 上运行以下命令：

      .. code-block:: bash

          allssh 'modify_firewall -f -r recovery_cvm_ip,recovery_virtual_ip -p 2030,2036,2073,2090 -i eth0'

      将 `remote_cvm_ip` 替换为主集群CVM的IP地址，用逗号隔开。

      将 `remote_virtual_ip` 替换为恢复集群的虚拟 IP 地址。

   #. 要打开与主集群通信的端口，请通过 SSH 远程连接（例如 ssh nutanix@<CLUSTER-VIRTUAL-IP>）在 *Recovery* 站点集群的 CVM 上运行以下命令：

      .. code-block:: bash

         allssh 'modify_firewall -f -r primary_cvm_ip,primary_virtual_ip -p 2030,2036,2073,2090 -i eth0'

      将 `source_cvm_ip` 替换为主集群 CVM 的 IP 地址，以逗号分隔。

      用主集群的虚拟IP地址替换 `source_virtual_ip` 。

   #. 退出两个SSH会话。

   .. note::

      有关所需端口的更多信息，请参阅`Xi Leap Administration Guide <https://portal.nutanix.com/page/documents/details?targetId=Leap-Xi-Leap-Admin-Guide-v5_19:Leap-Xi-Leap-Admin-Guide-v5_19>`_中的 **General Requirements of Leap** .

- 此自动化脚本将自动部署 **10** 个Fiesta应用程序实例:

   - **User01-MYSQL-...**; **User01-WebServer-...**
   - **User02-MYSQL-...**; **User02-WebServer-...**
   - 等等...

- 讲师已经每位参与者分配一个 *Userxx* 编号。 实验指南将使用 *UserXX* 引用实体名称，该名称应替换为它们的特定编号（例如 *User01*）。

- 建议将 Prism 中的集群分别重命名为 *Primary* 和 *Recovery*。 这将有助于在实验期间进行识别。

参考
+++++++++

以下资源仅供参考，不是完成实验练习所必需的。

- `Xi Leap Administration Guide <https://portal.nutanix.com/page/documents/details?targetId=Leap-Xi-Leap-Admin-Guide-v5_19:Leap-Xi-Leap-Admin-Guide-v5_19>`_
