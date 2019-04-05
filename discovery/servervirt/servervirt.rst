.. _servervirt:

------------------------------
General Server Virtualization
------------------------------

Overview
--------

When planning to migrate or implement new workloads on Nutanix, it is important to separate and understand applications, as different applications may have different sizing considerations.

General server virtualization is typically used to refer to any workload that is not part of a well understood application with prescriptive sizing guideline and best practices. Middleware and application servers responsible for presenting data are common examples of general server workloads.

Sizing general server virtualization workloads for Nutanix is common for customers with datacenter consolidation projects or simply looking to refresh compute/storage for their existing environments.

Basic Discovery
----------------

Most of the key metrics required to size a Nutanix cluster for general server workloads can be obtained for the customers existing environment using tools such as :ref:`rvtools`.

- **Source Hypervisor** - The current hypervisor hosting VMs will determine which tools you are able to use to perform data collection:

  - vSphere - :ref:`rvtools`
  - Hyper-V - `Hyper-V Reporting Script <https://gallery.technet.microsoft.com/scriptcenter/Hyper-V-Reporting-Script-4adaf5d0>`_
  - XenServer - `Export Resource Data in XenCenter <https://docs.citrix.com/en-us/xencenter/6-5/xs-xc-pools/xs-xc-pools-export-data.html>`_

- **Workload Categorization** - Especially when using data collection tools, it's critical to review output with the prospect and separate any specialized applications from the rest of the data. For example:

  - **Mission Critical Database VMs** - These VMs may be pinned to certain hosts and require different vCPU:pCore oversubscription than the rest of the mixed workloads.
  - **"Monster" VMs** - Typically VMs with > 8 vCPUs and LOTS of RAM. These VMs should be given additional consideration in terms of placement and performance.
  - **Powered Off VMs/Templates** - The storage capacity consumed by template images should be factored into the overall consumed storage capacity. If there is a large number of Powered Off VMs that are **not** Templates, you should follow up with the prospect to understand the roll of these VMs. Are they used as Templates and hence should be counted towards storage capacity? Are they being decommissioned and shouldn't be accounted for in the sizing at all? Are they going to be migrated and powered on in the future and should be accounted for in the sizing?
  - **Virtual Desktops VMs** - In smaller environments there may be non-server VMs running alongside server workloads. These virtual desktops are best discovered/sized separately, even if the end result is to include all workloads on the same cluster.
  - **Unified Communications VMs** - Solutions like Avaya and Cisco UC have particular sizing and placement requirements and should be discovered/sized separately.

- **Number of VMs** - On a per cluster basis.

- **Total vCPUs** - On a per cluster basis. CPU utilization is only helpful if historical data is available, do **not** base utilization on individual point-in-time data captures.

- **Total Assigned Memory** - On a per cluster basis. As situations dependent on memory oversubscription are rare, we will assume each VM is capable of 100% utliizing the memory assigned to it.

- **Virtual Disk Format** - The Nutanix Storage Fabric does not write zeroes. All virtual disks are Thin Provisioned. However, if the hypervisor (e.g. vSphere) creates VMs using Thick Provisioned disks, the Nutanix Storage Container will honor that space reservation. If a customer is using Thick Provisioned disks in their environment today, they need to be made aware that to take full advantage of the underlying storage that their virtual disks will need to be converted to Thin Provisioned when migrating to Nutanix.

- **Consumed Storage Capacity** - On a per cluster basis. This is the amount of data VMs have consumed, **not** the amount of storage that has been provisioned overall.

- **Provisioned Storage Capacity** - On a per cluster basis. There may be a large discrepancy between amount of capacity provisioned vs. capacity consumed. While the prospect may only require a small amount of storage capacity today, it's still important to understand how much storage capacity they anticipate requiring for the initial project. It's also important that the prospect understand there is no technical requirement for them to over-buy/over-provision and that additional nodes/capacity can be easily added when needed.

- **Virtual Machine Operating Systems** - While this item doesn't directly impact sizing, it's still a critical discovery question especially when positioning AHV. Identify any legacy operating systems, hypervisor agents, or virtual appliances that may be in use and not yet supported on AHV/KVM.

- **Current Host Specifications** - On a per cluster basis. Understanding the prospect's current host counts, CPU models and memory configurations can help guide Nutanix node configuration and sizing density.

- **Current Storage Specifications** - On a per cluster basis. This item may not directly impact Nutanix sizing, but is helpful to gather at this stage as it can be used to help position the proposed Nutanix solution in terms of performance, operational advantages, etc.

Active Working Set
..................

When sizing a hybrid (SSD + HDD) cluster, the active working set, also referred to as hot data, should fit within the SSD tier to ensure fast and consistent performance.

Always ask if the prospect's current storage platform(s) offer any reporting on hot data/working set. If historical data on working set is unavailable, apply the following rule of thumb:

Using the prospect's full monthly backup data, take the delta between the last two months of full backups and multiply that value by 3.

For example:

  - January Full Backup - *10TiB*
  - February Full Backup - *12TiB*
  - Delta - *(12TiB - 10TiB) = 2TiB*
  - Working Set Estimate - *(2TiB \* 3) =* **6TiB**

.. note::

  - Large amounts of deletions during the month can result in a smaller full backup size and could lead to undersizing the SSD tier of a cluster. **To prevent this, review multiple months of full backups and use the average delta. Alternately, if the prospect has daily incremental backup data, you could sum the daily incrementals for a month and then multiply by 2-3.**

  - Multiplying by 3 is a *conservative estimate* based on a 70% read, 30% write workload (70/30 = 2.33). **If the prospect anticipates higher read percentages the multiplier may need to be increased to prevent undersizing SSD (e.g. 80/20 = 4x multiplier).**

`Credit: Josh Odgers <http://www.joshodgers.com/2014/09/25/rule-of-thumb-sizing-for-storage-performance-in-the-new-world/>`_

.. _servervirt_dr:

Disaster Recovery
.................

Business continuity planning is traditionally a complex space, but sizing a Nutanix solution to account for compute and storage requirements is defined by a small number of key metrics:

- **Recovery Point Objective (RPO)** - RPO is the maximum targeted period in which data might be lost from a service or application due to an outage. RPO can also be referred to as the frequency with which snapshots are taken of the environment.

- **Daily Change Rate (%)** - The change rate is the percentage of new or overwritten data that will be created every 24 hours.

- **Retention** - Retention defines how long hourly, daily, weekly, and monthly snapshots are stored. Different applications or groups of VMs may have different retention plans. Also if replicating to one or more remote sites, the remote site retention may differ from the local site retention.

.. note::

  Using these three metrics it is simple to estimate snapshot storage capacity requirements for a secondary site:

  *<Virtual Disk Total> + ( <Virtual Disk Total> \* <Daily Change Rate %> \* <Number of Snapshots to Retain> )*

  Nutanix Sizer is capable of automatically computing snapshot capacity requirements for both the local and remote site.

- **Number of VMs** - Not all VMs within an environment may require replication. The total number of VMs requiring either remote backup or remote disaster recovery will help right-size storage and compute for any remote clusters used as a target for replication.

- **Recovery Time Objective (RTO)** - While RTO doesn't directly impact sizing, it can have an overall impact on the solution proposed. For example, an aggressive RTO could lead to implementing a solution such as Metro Availability for DR.

- **Disaster Recovery Software** - What software is the customer currently using for runbook automation for DR (e.g. Site Recovery Manager, Zerto, etc.). Third party DR orchestration software shouldn't directly impact sizing, but is an important consideration in the overall solution.

- **Backup Software** - Similar to DR software, backup software shouldn't directly impact sizing but is important to understand for the overall solution. Certain backup solutions that integrate with Nutanix Change Block Tracking (CBT), such as HYCU, offer the ability to leverage Nutanix native snapshots on the cluster in addition to recovery points stored on external targets, consideration should be given to how many local Nutanix snapshots will be stored.

Additional Resources
--------------------

Refer to `Nutanix Portal Solution Documentation <https://portal.nutanix.com/#/page/solutions>`_ for additional Reference Architecture, Best Practice Guide, Tech Note, and Solution Note documents.
