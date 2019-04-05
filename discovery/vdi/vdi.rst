.. _vdi:

---
VDI
---

Overview
--------

Virtual Desktop Infrastructure (VDI) is a popular workload for Nutanix due to its scale out nature, adding more desktops requires more compute, more storage capacity, and more IOPS.

Basic Discovery
---------------

- **Greenfield/Brownfield** - Existing, or "brownfield," VDI opportunities are generally the simplest to size, as the customer already has data from their existing environment using their images and applications. Net new, or "greenfield" VDI environments can still be sized prescriptively, but opportunities over 500 concurrent desktops should strongly consider performing an assessment of their existing desktop environment to assist in validating any assumptions.

  The most common tool used for pre-sales desktop assessments is `Liquidware Stratusphere FIT <http://www.liquidware.com/products/stratusphere-fit>`_. Desktop assessment data for large greenfield VDI opportunities can prevent significant undersizing or oversizing of infrastructure and help prevent a lengthy pilot to determine host/desktop density.

- **Broker** - Determine which brokering solution the prospect is using, typically either Citrix XenDesktop or VMware Horizon View.

- **Desktop Profile** - For greenfield deployments a prospect may not be able to provide precise virtual desktop VM specifications such as required vCPU, RAM, and vCPU consolidation ratio. While an assessment or pilot are the ideal means of sizing based on the prospect's environment, mapping prospect workloads to well understood user profiles is also common practice.

  Nutanix Sizer offers multiple default profiles that map to LoginVSI workloads. LoginVSI is a load generation tool used across many infrastructure vendors to determine desktop performance and per host density. A breakdown of default LoginVSI profiles can be found `here <https://www.loginvsi.com/documentation/index.php?title=Login_VSI_Workloads>`_.

  Refer to the Sizer UI for the latest specifications used for each profile.

- **VM Specifications** - For brownfield deployments a prospect should be able to provide their current per VM vCPU and RAM requirements, as well as what vCPU:pCore consolidation ratio is on their existing hosts. If the prospect intends on upgrading their OS (e.g. Windows 7 -> Windows 10) or Office (e.g. Office 2010 -> 2016) as part of the migration to Nutanix this could significantly impact density, be sure to capture this information.

- **Desktop Type** - Are the desktops persistent or non-persistent? If planning for a mix of both, these would be entered as separate workloads in Sizer but could be accomodated within the same cluster.

  - **Persistent** - Similar to a traditional desktops, persistent virtual desktops persist changes to the VM across reboots. Persistent desktops can be VMs provisioned either as full clones of a master image, thin clones that share a base image for reads and independently store writes, or simply from disparate VMs. Persistent desktops offer the greatest flexibility to the user, including the ability to install their own applications and customize OS and application settings without dependence on a remote profile. Once the VM has been provisioned it will require patching/updating through traditional means such as SCCM, WSUS, or other 3rd party patch management tools, limiting the operational benefit of implementing VDI. Persistent desktops have greater storage capacity requirements compared to non-persistent desktops, but this can be mitigated through storage deduplication due to the significant overlap in VM data (application patches, Windows updates, etc.).
  - **Non-persistent** - Unlike persistent desktops, non-persistent desktops do not persist changes across VM reboots. Implementations of non-persistent desktops can vary based on the provisioning technology used, but in general, after a user session ends, the VM reverts back to a pristine state. Non-persistent desktops can simplify management operations as changes to the master image can be quickly rolled out (and rolled back) in a consistent fashion to large numbers of VMs. Non-persistent desktops can also eliminate the negative software and performance creep that can occur over time on traditional desktops or persistent virtual desktops. Due to the stateless nature of non-persistent desktops, this approach may not be viable for every use case. For use cases such as kiosks, which require no customization, non-persistent desktops are an ideal fit. For use cases where persisting end user customizations such as application settings are important, profile management solutions would need to be evaluated and employed. Similarly, the need to persist end user data would require additional network based storage.

- **OS & Office Versions** - Microsoft Windows and Office versions can have an impact CPU and memory consumption.

- **Anti-virus** - Understanding target anti-virus solution can also have an impact on density. Solutions such as McAfee MOVE and other hypervisor-level anti-virus solutions offer greater density through reduced CPU utilization compared to traditional agent-based anti-virus solutions.

- **Total Concurrent Users** - What is the total number of users who will be connected at any time? This primarily drives the CPU and memory sizing.

- **Total Named Users** - What is the total number of users who can access the environment? It is common for the number of named users to exceed the number on concurrently connected users. For example, to deploy a call center that operates three shifts you may have 500 virtual desktops serving 1500 users.

- **User Data/Profile** - Will the user profiles/user data be stored on the Nutanix cluster? Using AFS? How much data will be allocated on average to each user? The amount of storage required for user data would be based off of Named users, not Concurrent.

- **Gold Image Specifications** - For non-persistent desktops, how many gold images will be used? What size is the gold image? For persistent desktops, what size is the virtual disk being assigned to each user? Storage capacity isn't typically a constraint for non-persistent desktops, but should be a key consideration for persistent desktops.

- **Provisioning** - See :ref:`vdi_xd` and :ref:`vdi_view`

- **vGPU** - Do you currently use any graphics acceleration for your applications today? Can you provide a list of applications that are GPU dependent and what GPUs are being used? Can you provide details on the number and resolution of monitors used for these virtual desktops?

  NVIDIA GRID vGPU technology allows a supported hypervisor (AHV/AOS 5.5+, vSphere 6.0+, XenServer 7.1+) to split the resources of a GPU among multiple VMs. Framebuffer, or video RAM, is dedicated to individual VMs, and cores are shared across all VMs mapped to a single, physical GPU. The experience from profile to profile can be subjective, so POCs may be required to identify the ideal vGPU profile for a given application/user group. The key metrics that define a vGPU profile are the maximum number of monitors supported, the maximum resolution of each monitor, and the amount of framebuffer assigned.

  A full breakdown of vGPU profiles can be found under 1 Introduction to NVIDIA GRID > 1.3 Supported GPUs > 1.3.1 Virtual GPU Types in the `NVIDIA GRID vGPU User Guide <http://images.nvidia.com/content/grid/pdf/GRID-vGPU-User-Guide.pdf>`_.

  .. note::

    Using NVIDIA GRID vGPU requires additional licensing from NVIDIA not sold by Nutanix. Ensure your prospect and partner are aware of the requirement. Full details on vGPU licensing can be found `here <http://images.nvidia.com/content/pdf/grid/guides/GRID-Packaging-and-Licensing-Guide.pdf>`_.

- **IOPS** - IOPS is a legacy consideration for sizing storage for VDI solutions. As long as the SSD tier is sized appropriately per node via Sizer, IOPS should not be a consideration for sizing virtual desktops.

.. _vdi_xd:

Citrix XenDesktop
.................

- **Provisioning** - Which Citrix technology is going to be used to provision desktops?

  - **Machine Creation Services (MCS)** - MCS is a VM creation/orchestration framework installed as part of the broker (Desktop Delivery Controller) role and managed directly through Citrix Studio. Using MCS it is possible to share a single gold image to provision hundreds of virtual desktops, significantly minimizing the amount of storage capacity required. For greenfield opportunities, MCS should be the recommended provisioning technology due to its ease of use and ability to rely on Nutanix storage to deliver required performance. MCS supports both persistent and non-persistent desktop types. MCS is supported on all Nutanix-supported hypervisors.

  - **Provisioning Services (PVS)** - PVS is a Windows Server based workload separate from the XenDesktop broker that streams a VMs virtual disk over the network. Using PVS it is possible to share a single gold image to provision hundreds of virtual desktops, significantly minimizing the amount of storage capacity required. PVS predates MCS and is common in large scale, brownfield XenDesktop environments as it was able to optimize storage performance by caching reads in RAM, and later buffering desktop writes in VM memory.  If using PVS for a brownfield environment it is important to understand whether or not the PVS servers will run on Nutanix and size to accomodate as necessary. PVS was designed to deliver non-persistent desktops. PVS is supported on all Nutanix-supported hypervisors.

  To learn more about MCS vs. PVS on Nutanix, watch Getting up close and personal with MCS and PVS from Citrix Synergy:

  .. raw:: html

    <iframe width="640" height="360" src="https://www.youtube.com/embed/p47JwwpUArQ?rel=0&amp;showinfo=0&amp;vq=hd1080" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe>

.. Citrix XenApp
.. .............

.. _vdi_view:

VMware Horizon View
...................

- **Provisioning** - Which VMware technology is going to be used to provision desktops?

  - **Linked Clones** - Linked Clones are used to deliver non-persistent desktops. Linked Clones depend on a Windows based service called Composer which is responsible for orchestrating cloning operations with vCenter. Using Linked Clones and Composer it is possible to share a single gold image, called a Replica, to provision hundreds of virtual desktops, significantly minimizing the amount of storage capacity required.

  - **Full Clones** - Full Clones are used to deliver persistent desktops. Nutanix integration with VAAI results in fast initial cloning of a master image, and deduplication can continue to keep the storage footprint small as individual VMs are patched, have applications installed, etc.

  - **Instant Clones** - Similar to Linked Clones, Instant Clones shared a parent VM virtual disk, consuming less storage capacity than Full Clones. Additionally, Instant Clones share the memory of the parent VM, with the goal being faster provisioning of non-persistent desktops through VMware vmFork technology. While currently not as flexible as Linked Clones, Instant Clones offer less administrative overhead compared to Linked Clones and do not depend on Composer.

Disaster Recovery
.................

Does the prospect plan to have their virtual desktops accessible from multiple datacenters?

VDI environments are not like protecting traditional datacenter applications, there is little to replicate and orchestrate at the storage/hypervisor layer in the event of an outage. Typically separate sites will have virtual servers for brokering independent of one another, allowing a load balancer to direct users to the most appropriate site.

The exceptions are:

  - **Gold Images** - Even for non-persistent desktops, it's important to ensure gold images are being replicated across sites.
  - **Persistent Desktops** - Do they plan on having DR capabilities for persistent desktop users? Or will those users fall back to a non-persistent desktop? It is important to consider the storage capacity requirements when providing DR for persistent desktops.
  - **User Data** - Where do users profiles and data live? How is that being replicated to the secondary site? If using AFS, PeerSync provides DFS-R capabilities to replicate user data to multiple sites.

Additional Resources
--------------------

`XenApp on AHV Reference Architecture <https://portal.nutanix.com/#/page/solutions/details?targetId=RA-2077-Citrix-XenApp-on-AHV:RA-2077-Citrix-XenApp-on-AHV>`_

`XenApp on vSphere Reference Architecture <https://portal.nutanix.com/#/page/solutions/details?targetId=RA-2003_Citrix_XenApp_on_vSphere:RA-2003_Citrix_XenApp_on_vSphere>`_

`XenDesktop on AHV Reference Architecture <https://portal.nutanix.com/#/page/solutions/details?targetId=RA-2035_Citrix_XenDesktop_on_AHV:RA-2035_Citrix_XenDesktop_on_AHV>`_

`XenDesktop on vSphere Reference Architecture <https://portal.nutanix.com/#/page/solutions/details?targetId=RA-2022_Citrix_XenDesktop_on_vSphere:RA-2022_Citrix_XenDesktop_on_vSphere>`_

`XenDesktop on XenServer Reference Architecture <https://portal.nutanix.com/#/page/solutions/details?targetId=RA-2087-Citrix-XenDesktop-on-XenServer:RA-2087-Citrix-XenDesktop-on-XenServer>`_

`VMware Horizon 7 Reference Architecture <https://portal.nutanix.com/#/page/solutions/details?targetId=RA-2058-VMware-Horizon:RA-2058-VMware-Horizon>`_

Refer to `Nutanix Portal Solution Documentation <https://portal.nutanix.com/#/page/solutions>`_ for additional Tech Note and Solution Note documents.
