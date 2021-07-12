.. _onpremleap2_UPFO:

----------------------------
Unplanned Failover with Leap
----------------------------

There are 3 types of failovers: Test, Planned and Unplanned.

- **Test failovers** are for testing a recovery plan. VMs are started in the test network as specified in the recovery plan. VMs at the primary location are not affected.

- **Planned failovers (PFO)** are when disruption of services is predicted at the primary site. The recovery plan will first create a snapshot of each VM, replicate, then start them at the recovery location. Those VMs no longer run at the primary site after a planned failover has occurred. Replication then begins in the reverse direction (from *Recovery* site to *Primary* site).

- **Unplanned failovers (UPFO)** occur when a disaster has already occurred at the primary location. VMs are recovered from the most recent snapshot, and are started at the *Recovery* site.

In this exercise you will perform an **Unplanned** failover of your application.

.. note::

   If you have already completed the :ref:`onpremleap3_PFO` exercise, you can skip to `Performing An Unplanned Failover`_.

Instructor Lead
+++++++++++++++

.. raw:: html

   <strong><font color="red">The following steps should be performed by either the instructor or a designated user, as enabling Leap and configuring the Availability Zone are one-time operations per Prism Central instance.</font></strong>

Enable Leap
...........

#. Within your *Primary* site Prism Central, select :fa:`bars` **> Prism Central Settings**.

#. Within the *Setup* section, click **Enable Leap > Enable**.

#. Within *Recovery* site Prism Central, select :fa:`bars` **> Prism Central Settings**.

#. Within the *Setup* section, click **Enable Leap > Enable**.

Creating a New Availability Zone
................................

#. Within *Primary* site Prism Central, select :fa:`bars` **> Administration > Availability Zones**, and observe that a Local AZ has already been created by default.

#. Click **Connect to Availability Zone**

   .. figure:: images/AZ/1.png

#. In the *Availability Zone Type* drop-down, select **Physical Location**. Enter the IP, username, and password for the *Recovery* site PC, and click **Connect**.

   .. figure:: images/AZ/2.png
      :align: left

   .. figure:: images/AZ/3.png
      :align: right

#. Observe that the *Recovery* site cluster is now listed as *Physical*, and its *Connectivity Status* is listed as *Reachable*.

Staging Guest Script
++++++++++++++++++++

Leap allows you to execute scripts within a guest to update configuration files, or perform other critical functions, as part of a runbook. In this exercise, you'll utilize a script on your WebServer VM that will automatically update its configured IP information for the MySQL VM connection. This allows the WebServer to connect to the MySQL database after any failover or failback operation.

.. raw:: html

   <strong><font color="red">The below script has already been deployed, as Calm allows for easily inserting steps (such as this script) at any point during the deployment of a blueprint.

   The following steps are included for illustration purposes only.</strong></font>

|

   - SSH into your *UserXX*\ **-WebServer** VM using the following credentials:

      - **User Name** - centos
      - **Password**  - nutanix/4u

   - Within the SSH session, execute the following commands. Click the icon in the upper right-hand corner of the below window to copy the commands to your clipboard. You may then paste that within your SSH session.

      .. code-block:: bash

         sudo wget -O /usr/local/sbin/production_vm_recovery https://raw.githubusercontent.com/nutanixworkshops/leap_addon_bootcamp/master/production_vm_recovery
         sudo chmod +x /usr/local/sbin/production_vm_recovery

      .. note::

         If you'd like to view the contents of the failover script, execute:

         ``sudo cat /usr/local/sbin/production_vm_recovery``

   - You may now exit the SSH session.

Installing Nutanix Guest Tools
++++++++++++++++++++++++++++++

In order to take advantage of the guest script functionality, Nutanix Guest Tools (NGT) must first be installed within the guest VMs being protected.

#. Within *Primary* site Prism Central, open :fa:`bars` **> Virtual Infrastructure > VMs**.

#. Select both your *UserXX*\ **-WebServer** and *UserXX*\ **-MySQL** VMs.

#. Click **Actions > Install NGT**. You may need to scroll down in the drop-down list.

   .. figure:: images/22.png

#. Select **Restart as soon as the install is completed**, then click **Confirm & Enter Password**.

   .. figure:: images/23.png

#. Provide the following credentials, and then click **Done** to begin the NGT installation:

   - **User Name** - centos
   - **Password**  - nutanix/4u

   .. figure:: images/24.png

#. Once both VMs have rebooted, validate that both VMs now have empty CD-ROM drives, and **Installed Version** displays **Latest** in Prism Central.

   .. figure:: images/25.png

Creating A Protection Policy
++++++++++++++++++++++++++++

A protection policy is where you specify your Recovery Point Objectives (RPO) and retention policies.

#. Within *Primary* site Prism Central, select, open :fa:`bars` **> Policies > Protection Policies**.

#. Click **Create Protection Policy**.

#. Within the **Policy name** field, enter *UserXX*\ **-FiestaProtection**.

#. Fill out the following fields within the *Primary Location* section, and then click **Save**.

   - **Location** - `Local AZ`
   - **Cluster** - Primary

#. Fill out the following fields within the *Recovery Location* section, and then click **Save**.

   - **Location** - `PC_<RECOVERY-SITE-PC-IP>`
   - **Cluster** - Recovery

#. Click **+ Add Schedule**. Select **Synchronous > Save Schedule**, and then click **Next**.

#. Click **Create**.

   .. note::

      While we are not demonstrating this method, protection policies can be automatically applied based on category assignment, allowing VMs to be automatically protected from their initial provisioning.

   .. figure:: images/Protection/protect1.png

#. Within *Primary* site Prism Central, open :fa:`bars` **> Virtual Infrastructure > VMs**.

#. Select both your *UserXX*\ **-WebServer** and *UserXX*\ **-MySQL** VMs.

#. Click **Actions > Protect**.

#. Select your *UserXX*\ **-FiestaRecovery** protection policy, and then click **Protect**.

   .. figure:: images/Protection/protect2.png

Creating A Recovery Plan
++++++++++++++++++++++++

.. note::

   Just as with Protection Policies, you can also add categories to any protection policy.

#. Within your *Primary* site Prism Central, open :fa:`bars` **> Policies > Recovery Plans**.

#. Click **Create New Recovery Plan**.

#. Fill out the following fields within the *General* section, and then click **Next**.

   - **Recovery Plan Name** - *UserXX*\ **-FiestaRecovery**\
   - **Recovery Plan Name** - (optional)
   - **Primary Location** - Local AZ
   - **Recovery Location** - `PC_<RECOVERY-SITE-PC-IP>`

   .. figure:: images/Recovery/1.png

.. note::

   If you do not see your VMs, the synchronization hasn't completed between sites. This is typically caused by attempting this step before the replication completes, but could indicate a communication issue between clusters. Check Prism Central for any errors, and if you are seeing issues initiating the stretch cluster, revisit the initial firewall instructions, and ensure those steps were performed correctly.

#. Under **Power On Sequence** we will add our VMs in stages to the plan. Click **+ Add Entities**.

#. Select your *UserXX*\ **-MySQL** VM, and then click **Add**.

#. Click **+ Add New Stage**. Within **Stage 2**, click **+ Add Entities**.

   .. figure:: images/Recovery/3.png

#. Select your *UserXX*\ **-WebServer** VM, and then click **Add**.

   .. figure:: images/Recovery/4.png

#. Click **Add**.

#. Select your *UserXX*\ **-WebServer** VM, and click **Manage Scripts > Enable**. This will trigger the *production_vm_recovery* script to run within the guest VM whenever a failover or failback occurs.

#. Click the **+ Add Delay**, shown in-between your two stages.

   .. figure:: images/Recovery/5.png

#. Specify **60** seconds, and then click **Add**.

#. Click **Next**.

   In the following steps, you will configure network settings which enable you to map networks in the local availability zone (*Primary* site), to networks at the recovery location (*Recovery* site).

#. Click **OK. Got it**.

#. Select **Primary** for all *Virtual Network or Port Group* entries.

   .. figure:: images/Recovery/6.png

#. Click **Done**.

   .. note::

      Leap guest script locations

         - **Windows** (Relative to Nutanix directory in Program Files)

            Production: scripts/production/vm_recovery.bat

            Test: scripts/test/vm_recovery.bat

         - **Linux**

            Production: /usr/local/sbin/production_vm_recovery

            Test: /usr/local/sbin/test_vm_recovery

Performing An Unplanned Failover
++++++++++++++++++++++++++++++++

Failovers are initiated from the remote site, which can either be another on-premises Prism Central located at your DR site, or Xi Cloud Services.

In this exercise, we will be connecting to an on-premises Prism Central at the *Recovery* site, which we've already paired with the *Primary* site on-prem cluster.

Ensure that the VM name doesn't exist on the *Primary* cluster.

Before performing our failover, let's make a quick update to our application.

#. Open `<http://USERXX-WEBSERVER-IP-ADDRESS>`_ in another browser tab. (ex. `<http://10.42.212.50>`_)

#. Under **Stores**, click **Add New Store** and fill out the required fields. Validate your new store appears in the UI.

   .. figure:: images/Failover/1.png

#. Log in to Prism Central for your *Recovery* site.

#. Open :fa:`bars` **> Policies > Recovery Plans**.

#. Select your *UserXX*\ **-FiestaRecovery** plan, and then click **Actions > Failover**.

#. Under **Failover Type**, select **Unplanned Failover**, and then click **Failover**.

   .. figure:: images/Failover/2.png

#. Ignore any warnings in the Recovery AZ (*Recovery* site), and then click **Execute Anyway**.

#. Click on *UserXX*\ **-FiestaRecovery** to monitor status of plan execution. Select **Tasks > Failover** for full details.

   .. figure:: images/Failover/4.png

   .. note::

      If you had validation warnings before initiating failover, it is normal for the *Validating Recovery Plan* step to show a status of *Failed*.

#. Once the Recovery Plan reaches 100%, click on the **X** at the top right-hand corner. This will take approximately 5 minutes.

#. Open :fa:`bars` **> Virtual Infrastructure > VMs**, and note the *Recovery* site IP address of your *UserXX*\ **-WebServer**.

#. Open `<http://USERXX-WEBSERVER-VM-RECOVERYSITE-IP-ADDRESS>`_ (ex. `<http://10.42.212.50>`_) in another browser tab and verify the change you'd made to your application is present.

Congratulations! You've completed your first DR failover with Nutanix AHV, leveraging native Leap runbook capabilities and synchronous replication.

Performing An Unplanned Failback
++++++++++++++++++++++++++++++++

Before performing our failback, let's make another update to our application.

#. Return to the browser tab for `<http://USERXX-WEBSERVER-VM-RECOVERYSITE-IP-ADDRESS>`_ (ex. `<http://10.42.212.50>`_).

#. Under **Stores**, click **Add New Store**, and then fill out the required fields. Validate your new store appears in the UI.

   .. figure:: images/Failover/1.png

#. Log in to Prism Central for your *Primary* site.

#. Open :fa:`bars` **> Virtual Infrastructure > VMs**.

#. Select both of your VMs, and then click **Actions > Delete**. Confirm by clicking **Delete**.

#. Open :fa:`bars` **> Policies > Recovery Plans**.

#. Select your *UserXX*\ **-FiestaRecovery** plan, and then click **Actions > Failover**.

   .. figure:: images/Failover/2.png

#. Under **Failover Type**, select **Unplanned Failover**, and then click **Failover**.

   .. figure:: images/Failover/2.png

#. Ignore any warnings in the Recovery AZ (*Primary* site), and then click **Execute Anyway**.

#. Click the name of your Recovery Plan to monitor status of plan execution. Select **Tasks > Failover** for full details.

   .. figure:: images/Failover/4.png

.. note::

   If you had validation warnings before initiating failover, it is normal for the *Validating Recovery Plan* step to show a status of *Failed*.

#. Once the Recovery Plan reaches 100%, click on the **X** at the top right-hand corner. This will take approximately 5 minutes.

#. Open :fa:`bars` **> Virtual Infrastructure > VMs** and note the *Primary* site IP Address of your *UserXX*\ **-WebServer**.

#. Open `<http://USERXX-WEBSERVER-VM-PRIMARYSITE-IP-ADDRESS>`_ in another browser tab, and then verify the change you'd made to your application is present.

Congratulations! You've completed your first DR failback with Nutanix AHV, leveraging native Leap runbook capabilities, and synchronous replication.
