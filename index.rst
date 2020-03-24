.. title:: My Recovery Deep Dive

.. toctree::
  :maxdepth: 2
  :caption: My Recovery Deep Dive
  :name: _xileap
  :hidden:

  index

.. toctree::
  :maxdepth: 2
  :caption: Appendix
  :name: _appendix
  :hidden:

---------------------
My Recovery Deep Dive
---------------------

Legacy disaster recovery configurations, which are created with Prism Element, use protection domains and third-party integrations to protect VMs, and they replicate data between on- premises Nutanix clusters. Protection domains provide limited flexibility in terms of supporting operations such as VM boot order and require you to perform manual tasks to protect new VMs as an application scales up.

Leap uses an entity-centric approach and runbook-like automation to recover applications. It uses categories to group the entities to be protected and to automate the protection of new entities as the application scales. Application recovery is more flexible with network mappings, configurable stages to enforce a boot order, and optional inter-stage delays. Application recovery can also be validated and tested without affecting production workloads. All the configuration information that an application requires upon failover are synchronized to the recovery location.

You can use Leap between two physical data centers or between a physical data center and Xi Cloud Services. Leap works with pairs of physically isolated locations called availability zones. One availability zone serves as the primary location for an application while a paired availability zone serves as the recovery location. While the primary availability zone is an on-premises Prism Central instance, the recovery availability zone can be either on-premises or in Xi Cloud Services.

**In this lab you will explore Xi Leap, configuring a Protection Policy, building a Recovery Plan, evaluating networking considerations, and perform a failover.**

.. _accessingleaplab:

Accessing Your Lab Environment
++++++++++++++++++++++++++++++

This lab requires both a Nutanix Xi Leap cluster and a traditional on-premises Nutanix cluster.

You can access your environments by navigating to https://access.nutanixtestdrive.com in your browser after clicking the link in your e-mail.

In the table, look for **BCDR Deep Dive**.

- The **Launch** button will connect you to the On-Prem Prism Central
- The **Xi** button will connect you to the Xi Leap side.

.. figure:: images/access_dashboard.png

You can also access manually by the following URLs:

- On-Prem Prism Central: td**xxxxxxxxxx**.nutanixtestdrive.com/console/ (where **xxxxxxxxxx** is specific to your environment)
- Xi Leap: td**xxxxxxxxxx**-xi.nutanixtestdrive.com/xi/#page/xi_dashboard


Verifying Cluster Pairing
+++++++++++++++++++++++++

**Availability Zones** represent physically separate groups of resources, whether that be Nutanix clusters across different sites, or Xi Cloud Services environments. In order to replicate between Availability Zones, your source cluster must first be paired to the target Prism Central or Xi environment.

#. In your **On-Prem Prism Central**, click :fa:`bars` **> Administration > Availability Zones**.

   .. figure:: images/new.png

#. Note that your **On-Prem** cluster has already been paired with the **US-EAST-1A** cloud environment.

#. Optionally, click **Connect to Availability Zone** to see what details are required to pair a new cluster or Xi Cloud environment.

Creating Categories
+++++++++++++++++++

In **Prism Central**, a **Category** is a key value pair. Categories are assigned to entities (such as VMs, Networks, or Images) based on some criteria (Location, Production-level, App Name, etc.). Different policies can then be mapped to categories, applying to all VMs with the appropriate assigned values.

For example, you might have a Department category that includes values such as Engineering, Finance, and HR. In this case you could create one backup policy that applies to Engineering and HR and a separate (more stringent) backup policy that applies to just Finance. Categories allow you to implement a variety of policies across entity groups, and Prism Central allows you to quickly view any established relationships.

#. In your **On-Prem Prism Central**, click :fa:`bars` **> Virtual Infrastructure > Categories**.

   .. figure:: images/4.png

#. Select the **Demo** category and click **Actions > Update**.

   .. figure:: images/5.png

#. Observe the category already has 3 available values. Click the :fa:`plus-circle` icon beside the last value to add a new value.

   .. figure:: images/add_category.png

#. In the **Values** column type in **App-4** and click **Save**.

   .. figure:: images/6.png

   Now you'll need to assign your category to an entity.

#. Click :fa:`bars` **> Virtual Infrastructure > VMs**.

#. Take note of the IP addresses for the deployed VMs.

#. Click the checkbox next to the **App-4** VM and click **Actions > Manage Categories**.

   .. figure:: images/manage_categories.png

#. In the search field, specify **Demo: App-4** and click **Save** to apply the category to the VM.

   .. figure:: images/7.png

Updating Protection Policy
++++++++++++++++++++++++++

A **Protection Policy** defines the desired RPO and snapshot policy.

#. In your **On-Prem Prism Central**, click :fa:`bars` **> Policies > Protection Policies**.

#. Click the checkbox next to the existing policy, **AppPR**, and click **Actions > Update**.

   .. figure:: images/8.png

#. Observe the different options for remote and local snapshot retention:

   - **Linear**

      Implements a simple, linear retention scheme at both local and remote sites. If you set the retention number for a given site to n, the n most recent snapshots are retained at that site. For example, if the RPO is one hour and theretention number for the local site is 48, the 48 most recent snapshots are retained at any given time.

   - **Roll-up**

      Rolls up the oldest snapshots for the specified RPO interval into a single snapshot when the next higher interval is reached, all the way up to the retention period specified for a site. For example, if you select roll-up retention, set the RPO to one hour, and set the retention time at a site to one year, the twenty four oldest hourly backups at that site are rolled up into a single daily backup at the completion of every 24 hours, the seven oldest daily backups are rolled up into a single weekly backup at the completion of every week, the four oldest weekly backups are rolled up into a single backup at the completion of every month, and the twelve oldest monthly backups are rolled up into a single backup at the completion of every year. At the end of one year, that site has 24 of the most recent hourly backups, seven of the most recent daily backups, four of the most recent weekly backups, twelve of the most recent monthly backups, and one yearly backup. The snapshots that are used to create a rolled-up snapshot are discarded.

   Next, you'll include your new category in the Protection Policy, ensuring all **App-4** tagged VMs share this policy.

#. Click **Update** and specify **Demo: App-4** in the **Add Categories** dialog.

   .. figure:: images/9.png

#. Click **Save**.

Updating Recovery Plan
++++++++++++++++++++++

A **Recovery Plan** defines the runbook for a failover event, including power on sequences and network mappings.

#. In your **On-Prem Prism Central**, click :fa:`bars` **> Policies > Recovery Plan**.

#. Click the checkbox next to the existing policy, **AppRP**, and click **Actions > Update**.

   .. figure:: images/10.png

#. Click **Next**. Observe the existing stages for powering on VMs as part of the recovery plan.

#. Under **Power On Sequence**, click **Add New Stage** to add a 4th stage.

   .. figure:: images/11.png

#. To add a delay between two stages, click **Add Delay** between the two stages and specify a value in **Seconds**. Click **Add**.

#. Click **Add Entities** under your new stage.

   .. figure:: images/add_entities0.png

#. Add the **Demo: App-4** category by searching for the **Demo** category (press Enter to Search), then check the box and click **Add**.

   .. figure:: images/add_entities.png

#. Click **Next**.

   **Network Settings** enables you to map networks in the local availability zone (the primary location) to networks at the recovery location. When failover occurs and VMs are recovered at the recovery location, they are placed in the network that is mapped to their network on the primary location.

   .. figure:: images/12.png

   Local availability zone on the left. On the left, you specify the VM Networks used for both **Production** and **Test** failover events. These are mapped to corresponding **Production** and **Test** networks in Xi Cloud. If the recovery location is an on-premises availability zone, you would then specify the corresponding VM Networks for that site.

   Optionally, you can specify a gateway IP address and prefix. With Xi Cloud Services, you either select a subnet that you have created, or you can enter the gateway IP address and prefix length. If you specify a gateway IP address and prefix length, the recovery plan dynamically creates the subnet on failover, and it cleans up the dynamically created subnets after failback. This functionality is a key benefit of DR with Xi Leap, as each VM will be able to maintain its original IP address when failing over to Xi Cloud, drastically reducing runbook scripting to re-configure applications.

   You can also assign VMs **Floating IPs**. This will provide direct access to the VM from the public Internet. Upon failover the public IP would get assigned to the VM in a NAT configuration.

Exploring Xi Cloud Portal
+++++++++++++++++++++++++

#. Return to your Xi Cloud session using the Xi Leap link in :ref:`accessingleaplab`.

#. Take some time to explore the Xi Cloud interface, comparing it to your on-premises Prism Central experience.

#. Click **Explore > Virtual Private Clouds**.

   This is where you can can see your VPCs on Xi Cloud. If you click on one of **Production** or **Test** you can configure subnets, VPNS, etc.

#. Click on **Production**.

   You can create up to 100 different subnets and build policy-based routing rules between them.

#. Click on **Manage VPN**.

   .. figure:: images/manage_vpn.png

#. Click **Next** to view the configuration requirements required to configure a VPN.

   While networking can be the biggest challenge of any hybrid cloud deployment, Nutanix strives to simplify this process (and reduce unnecessary professional services costs) by providing automatic configuration options. Stepping through the automatic configuration wizard will also provide all of the commands required to complete the configuration of your on-premises environment.

#. Click the **X** in the top right corner to close the configuration wizard.

#. Click **Yes** if prompted.

#. Click the **X** in the top right corner to close the VPC settings screen.

Performing A Failover
+++++++++++++++++++++

Leap supports the following types of failover operations:

- **Test Failover**

  You perform a test failover when you want to test a recovery plan. When you perform a test failover, the VMs are started in the virtual network designated for testing purposes at the recovery location (a manually created virtual network on on-premises clusters and a virtual subnet in the Test VPC in Xi Cloud Services). However, the VMs at the primary location are not affected. Test failovers rely on the presence of VM snapshots at the recovery location.

- **Planned Failover**

  You perform planned failover when a disaster that disrupts services is predicted at the primary location. When you perform a planned failover, the recovery plan first creates a snapshot of each VM, replicates the snapshots at the recovery location, and then starts the VMs at the recovery location. Therefore, for a planned failover to succeed, the VMs must be available at the primary location. If the failover process encounters errors, you can resolve the error condition. After a planned failover, the VMs no longer run in the source availability zone.

  After failover, replication begins in the reverse direction. For a planned failover the MAC address will be maintained.

- **Unplanned Failover**

  You perform unplanned failover when a disaster has occurred at the primary location. In an unplanned failover, you can expect some data loss to occur. The maximum data loss possible is equal to the RPO configured in the protection policy or the data that was generated after the last manual backup for a given VM. In an unplanned failover, by default, VMs are recovered from the most recent snapshot. However, you can recover from an earlier snapshot by selecting a date and time. Any errors are logged but the execution of the failover continues.

  After failover, replication begins in the reverse direction.

In this exercise you will be performing a **Planned Failover**.

  .. note::

    You can perform an unplanned failover operation only if snapshots have been replicated to the recovery availability zone. At the recovery location, failover operations cannot use snapshots that were created locally in the past. For example, if you perform a planned failover from the primary availability zone AZ1 to recovery location AZ2 (Xi Cloud Services) and then attempt an unplanned failover from AZ2 to AZ1, recovery will succeed at AZ1 only if snapshots were replicated from AZ2 to AZ1 after the planned failover operation. The unplanned failover operation cannot perform recovery based on snapshots that were created locally when the entities were running in AZ1.

#. From **Xi Cloud**, select **Explore > Recovery Plans**.

#. Select your **AppRP** plan and click **Actions > Failover**.

   .. figure:: images/14.png

#. Note the number of entities to be failed over as part of the plan. Click **Failover**.

   .. figure:: images/15.png

#. Once the recovery plan updates to **Running**, click **AppRP** and select **Tasks** to view the current status. Continue to observe the failover, taking note of the different boot stages.

   .. figure:: images/16.png

#. Once the failover operation has completed, select **VMs** from the sidebar.

#. Note that the 4 VM's are now running on the Xi Cloud.

   .. figure:: images/18.png

Failing Back
++++++++++++

Not to be confused with failing up, failing back to the primary site, including the changes that took place while running at the recovery site, is a critical part of the DR workflow.

#. Return to your **On-Prem Prism Central** and click :fa:`bars` **> Policies > Recovery Plan**.

#. Select your **AppRP** plan and click **Actions > Failover**.

#. When failing back, your **Recovery Location** should correspond to your primary site. Click **Failover**.

#. Click on your recovery plan and note you have access to the same UI in Prism Central to monitor the failback operation.

#. Once the recovery plan has completed, validate the VMs are once again running.

Takeaways
+++++++++

- Xi Leap delivers fast and easy cloud DR capabilities to on-premises Nutanix clusters
- Xi Leap drastically simplifies networking requirements by providing automated VPN setup
- Having the same networks available in Xi Cloud drastically simplify VM runbooks
- Xi Leap supports test, planned, and unplanned failover operations
- Failback operations from Xi Leap require no special changes to your runbooks
- Prism Central and Xi Cloud provide a unified management experience for managing and monitoring DR operations
- Leap can also be used to deliver native DR capabilities between multiple on-premises Nutanix AHV clusters
