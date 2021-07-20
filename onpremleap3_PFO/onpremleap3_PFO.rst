.. _onpremleap3_PFO:

----------------------------
使用 Leap 进行计划故障转移
----------------------------

有 3 种类型的故障转移：测试、计划和计划外。

- **Test failovers** 用于测试恢复计划。 虚拟机按照恢复计划中的规定在测试网络中启动。 主站点的虚拟机不受影响。

- **Planned failovers (PFO)** 是在主站点预测服务中断。 恢复计划将首先创建每个虚拟机的快照、复制，然后在容灾站点启动虚拟机。 发生计划中的故障转移后，这些虚拟机不再在主站点上运行。 然后以相反的方向开始复制（从 *Recovery* 站点到 *Primary* 站点）。

- **Unplanned failovers (UPFO)** 当灾难已经在主要位置发生时发生。虚拟机从最近的快照恢复，且在 *Recovery* 站点启动。

在本练习中，您将对应用程序执行 **Planned** 故障转移。

.. note::

   如果你已经完成了 :ref:`onpremleap2_UPFO` 练习, 你可以跳到 `Performing A Planned Failover`_.

讲师指导
+++++++++++++++

.. raw:: html

   <strong><font color="red">以下步骤应由讲师或一个用户执行，因为启用 Leap 和配置可用区是每个 Prism Central 的一次性操作。
   </font></strong>

启用 Leap
...........

#. 在 *Primary* 站点的Prism Central中, 选择 :fa:`bars` **> Prism Central Settings**.

#. 选择 *Setup*d 选项, 点击 **Enable Leap > Enable**.

#. 在 *Recovery* 站点的 Prism Central上, 选择 :fa:`bars` **> Prism Central Settings**.

#. 选择 *Setup* 选项, 点击 **Enable Leap > Enable**.

创建新的Availability Zone
................................

#. 在 *Primary* 站点的 Prism Central上, 选择 :fa:`bars` **> Administration > Availability Zones**, 并观察到默认情况下已经创建了Local AZ。

#. 点击 **Connect to Availability Zone**

   .. figure:: images/AZ/1.png

#. 在 *Availability Zone Type* 选项下拉, 选择 **Physical Location**. 输入*Recovery* 站点 PC的IP地址, 用户名, 和密码 , 并点击 **Connect**.

   .. figure:: images/AZ/2.png
      :align: left

   .. figure:: images/AZ/3.png
      :align: right

#. 请注意， *Recovery* 站点集群现在列为 *Physical*, 其 *Connectivity Status* 列为 *Reachable*.

Staging Guest 脚本
++++++++++++++++++++

Leap 允许您在Guest中执行脚本以更新配置文件或执行其他关键功能，作为 Runbook 的一部分。 在本练习中，您将使用 WebServer VM 上的脚本，该脚本将自动更新 MySQL VM 连接的配置 IP 信息。 这允许 WebServer 在任何故障转移或故障回复操作后连接到 MySQL 数据库。

.. raw:: html

   <strong><font color="red">下面的脚本已经部署，因为 Calm 允许在蓝图部署期间的任何时候轻松插入步骤（例如这个脚本）。

   包含以下步骤仅用于说明目的。</strong></font>

|

- 使用以下口令SSH登录到你的 *UserXX*\ **-WebServer** 虚拟机:

   - **User Name** - centos
   - **Password**  - nutanix/4u

- 在 SSH 会话中，执行以下命令。 单击下面窗口右上角的图标，将命令复制到剪贴板。 然后，您可以将其粘贴到 SSH 会话中。

   .. code-block:: bash

      sudo wget -O /usr/local/sbin/production_vm_recovery https://raw.githubusercontent.com/nutanixworkshops/leap_addon_bootcamp/master/production_vm_recovery
      sudo chmod +x /usr/local/sbin/production_vm_recovery

   .. note::

      如果您想查看故障转移脚本的内容，请执行:

      ``sudo cat /usr/local/sbin/production_vm_recovery``

- 你现在可以退出SSH会话。

安装 Nutanix Guest Tools
++++++++++++++++++++++++++++++

为了利用Guest脚本功能，必须首先在受保护的Guest VM 中安装 Nutanix Guest Tools (NGT)。

#. 在 *Primary* 站点的Prism Central中, 打开 :fa:`bars` **> Virtual Infrastructure > VMs**.

#. 选择两个你的 *UserXX*\ **-WebServer** 和 *UserXX*\ **-MySQL** VMs.

#. 点击 **Actions > Install NGT**. 您可能需要在下拉列表中向下滚动.

   .. figure:: images/22.png

#. 选择 **Restart as soon as the install is completed**, 然后点击 **Confirm & Enter Password**.

   .. figure:: images/23.png

#. 输入以下凭证, 然后点击 **Done** 开始 NGT 的安装:

   - **User Name** - centos
   - **Password**  - nutanix/4u

   .. figure:: images/24.png

#. 一旦两个 VM 都重新启动，验证两个 VM 现在都有空的 CD-ROM 驱动器，并且 **Installed Version** 在Prism Central中显示为 **Latest** 。

   .. figure:: images/25.png

创建保护策略
++++++++++++++++++++++++++++

保护策略是您指定恢复点目标 (RPO) 和保留策略。

#. 在 *Primary* 站点的Prism Central中, 选择, 打开 :fa:`bars` **> Policies > Protection Policies**.

#. 点击 **Create Protection Policy**.

#. 在 **Policy name** 字段中, 输入 *UserXX*\ **-FiestaProtection**.

#. 在 *Primary Location* 字段中填写以下字段, 然后点击 **Save**.

   - **Location** - `Local AZ`
   - **Cluster** - Primary

#. 在 *Recovery Location* 字段中填写以下字段, 然后点击 **Save**.

   - **Location** - `PC_<RECOVERY-SITE-PC-IP>`
   - **Cluster** - Recovery

#. 点击 **+ Add Schedule**. 选择 **Synchronous > Save Schedule**, 然后点击 **Next**.

#. 点击 **Create**.

   .. note::

      虽然我们没有演示这种方法，但可以根据类别分配自动应用保护策略，从而允许从初始配置中自动保护虚拟机。

   .. figure:: images/Protection/protect1.png

#. 在 *Primary* 站点的 Prism Central中, 打开 :fa:`bars` **> Virtual Infrastructure > VMs**.

#. 选择两个你的 *UserXX*\ **-WebServer** 和 *UserXX*\ **-MySQL** VMs.

#. 点击 **Actions > Protect**.

#. 选择你的 *UserXX*\ **-FiestaRecovery** 保护策略, 然后点击 **Protect**.

   .. figure:: images/Protection/protect2.png

创建恢复计划
++++++++++++++++++++++++

.. note::

   与保护策略一样，您也可以向任何保护策略添加类别。

#. 在 *Primary* 站点的 Prism Central中, 打开 :fa:`bars` **> Policies > Recovery Plans**.

#. 点击 **Create New Recovery Plan**.

#. 在 *General* 字段中填写以下字段, 然后点击 **Next**.

   - **Recovery Plan Name** - *UserXX*\ **-FiestaRecovery**\
   - **Recovery Plan Name** - (optional)
   - **Primary Location** - Local AZ
   - **Recovery Location** - `PC_<RECOVERY-SITE-PC-IP>`

   .. figure:: images/Recovery/1.png

.. note::

   如果您没有看到您的 VM，则站点之间的同步尚未完成。 这通常是由于在复制完成之前尝试此步骤造成的，但也有可能是集群之间存在通信问题。 检查 Prism Central 是否有任何错误，如果您在启动延展集群时遇到问题，请重新访问初始防火墙说明，并确保正确执行了这些步骤。

#. 在 **Power On Sequence** 下，我们将分阶段将你的虚拟机添加到计划中。点击 **+ Add Entities**.

#. 选择你的 *UserXX*\ **-MySQL** 虚拟机, 然后点击 **Add**.

#. 点击 **+ Add New Stage**. 在 **Stage 2** 中, 点击 **+ Add Entities**.

   .. figure:: images/Recovery/3.png

#. 选择你的 *UserXX*\ **-WebServer** 虚拟机, 然后点击 **Add**.

   .. figure:: images/Recovery/4.png

#. 点击 **Add**.

#. 选择你的 *UserXX*\ **-WebServer** 虚拟机, 然后点击 **Manage Scripts > Enable**. 每当发生故障转移或故障回复时，这将触发 *production_vm_recovery* 脚本在Guest VM中运行。

#. 点击 **+ Add Delay**, 显示在你的两个阶段之间。

   .. figure:: images/Recovery/5.png

#. 设为 **60** 秒, 然后点击 **Add**.

#. 点击 **Next**.

   在以下步骤中，您将配置网络设置，使您能够将本地availability zone(*Primary* site)中的网络映射到容灾站点 (*Recovery* site)的网络。

#. 点击 **OK. Got it**.

#. 为所有的 *Virtual Network or Port Group* 条目选择 **Primary** 。

   .. figure:: images/Recovery/6.png

#. 点击 **Done**.

   .. note::

      Leap guest 脚本位置

         - **Windows** (Relative to Nutanix directory in Program Files)

            Production: scripts/production/vm_recovery.bat

            Test: scripts/test/vm_recovery.bat

         - **Linux**

            Production: /usr/local/sbin/production_vm_recovery

            Test: /usr/local/sbin/test_vm_recovery for Windows and Linux guests.

Performing A Planned Failover
执行计划内故障转移
++++++++++++++++++++++++++++++++

故障转移是从远程站点启动的，远程站点可以是位于您的 DR 站点的另一个本地 Prism Central，也可以是 Xi 云服务。

在本练习中，我们将连接到 *Recovery* 站点的本地 Prism Central，我们已经将其与 *Primary* 站点本地集群配对。

确保 *Primary* 群集上不存在 VM 名称。

在执行故障转移之前，让我们快速更新我们的应用程序。

#. 在另外一个浏览器页面打开 `<http://USERXX-WEBSERVER-IP-ADDRESS>`_ 。 (例如 `<http://10.42.212.50>`_)

#. 在 **Stores**下, 点击 **Add New Store** 并填写必填字段。 验证您的新商店是否出现在 UI 中。

   .. figure:: images/Failover/1.png

#. 登录 *Recovery* 站点的 Prism Central。

#. 打开 :fa:`bars` **> Policies > Recovery Plans**.

#. 选择你的 *UserXX*\ **-FiestaRecovery** 计划, 然后点击 **Actions > Failover**.

#. 在 **Failover Type**下, 选择 **Planned Failover**, 然后点击 **Failover**.

   .. figure:: images/Failover/3a.png

   .. note::

      您可能想知道为什么我们不选中 *Live Migrate VMS* 框。 在我们的 HPOC 环境中，每个集群之间的 CIDR（例如 /25、/26）地址不同，这使我们无法在 HPOC 环境中使用此选项。

#. 忽略Recovery AZ (*Recovery* site)中的任何警告, 然后点击 **Execute Anyway**.

#. 点击 *UserXX*\ **-FiestaRecovery** 来监控计划执行的状态。选择 **Tasks > Failover** 以获取完整详细信息。

   .. figure:: images/Failover/4a.png

   .. note::

      如果您在启动故障转移之前收到验证警告，则 *Validating Recovery Plan* 步骤显示 *Failed* 是正常的。

#. 恢复计划达到 100% 后，单击右上角的 **X** 。 这将需要大约 5 分钟。

#. 打开 :fa:`bars` **> Virtual Infrastructure > VMs**, 并记下你的 *UserXX*\ **-WebServer** 的 *Recovery* 站点的IP地址.

#. 在另一个浏览器选项卡中打开 `<http://USERXX-WEBSERVER-VM-RECOVERYSITE-IP-ADDRESS>`_ (例如 `<http://10.42.212.50>`_) 并验证您所做的更改 您的应用程序存在。

Congratulations! You've completed your first DR failover with Nutanix AHV, leveraging native Leap runbook capabilities and synchronous replication.
恭喜！ 您已经使用 Nutanix AHV 完成了第一次灾难恢复故障转移，充分利用了本地 Leap Runbook 功能和同步复制。

执行计划内故障恢复
++++++++++++++++++++++++++++++++

在执行故障恢复之前，让我们对应用程序进行另一次更新。

#. 返回浏览器选项卡 `<http://USERXX-WEBSERVER-VM-RECOVERYSITE-IP-ADDRESS>`_ (例如 `<http://10.42.212.50>`_).

#. 在 **Stores**下, 点击 **Add New Store**, 然后填写要求的字段。 验证您的新商店是否出现在 UI 中。

   .. figure:: images/Failover/1.png

#. 登录你*Primary* 站点中的Prism Central.

#. 打开 :fa:`bars` **> Policies > Recovery Plans**.

#. 选择你的 *UserXX*\ **-FiestaRecovery** 计划, 然后点击 **Actions > Failover**.

   .. figure:: images/Failover/2.png

#. 在 **Failover Type**下, 选择 **Planned Failover**, 然后点击 **Failover**.

   .. figure:: images/Failover/3a.png

#. 忽略Recovery AZ (*Primary* site)中的任何警告, 然后点击 **Execute Anyway**.

#. 单击您的恢复计划的名称以监控计划执行的状态。 选择 **Tasks > Failover** 以获取完整详细信息。

   .. figure:: images/Failover/4a.png

.. note::

   如果您在启动故障转移之前收到验证警告，则 *Validating Recovery Plan* 步骤显示 *Failed* 状态是正常的.

#. 恢复计划达到 100% 后，单击右上角的 **X** 。 这将需要大约 5 分钟。

#. 打开 :fa:`bars` **> Virtual Infrastructure > VMs** 并且记下你的 *UserXX*\ **-WebServer** 的 *Primary* 站点IP地址.

#. 在另个一个浏览器选项卡中打开 `<http://USERXX-WEBSERVER-VM-PRIMARYSITE-IP-ADDRESS>`_ 然后验证您对应用程序所做的更改是否存在。

恭喜！ 您已经使用 Nutanix AHV 完成了第一次灾难恢复故障恢复，利用了原生 Leap Runbook 功能和同步复制。
