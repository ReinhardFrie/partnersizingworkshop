.. _ntnx:

-------------------------------
Nutanix Specific Considerations
-------------------------------

Overview
--------

Nutanix Sizer automates and simplifies the process of sizing Nutanix infrastructure for specific workloads, but it is still important to understand certain best practices when architecting a Nutanix solution.

Availability
------------

Replication Factor 2
....................

- RF2 clusters require a minimum of 3 nodes. While there are only two copies of data, a fully healthy cluster requires 3 copies of metadata.

  .. note::

    Single-Node and Two-Node clusters are an exception to this minimum.

    Two-Node clusters provide the same level of resiliency as a standard 3 node RF2 cluster but leverage an external Witness VM to arbitrate in the event of a node failure. Refer to the `Prism Web Console Guide <https://portal.nutanix.com/#/page/docs/details?targetId=Web-Console-Guide-Prism-v56:wc-cluster-two-node-c.html>`_ for guidelines and requirements for Two-Node clusters.

    Single-Node clusters provide the ability to run a limited number of user VMs with local resiliency. If an SSD fails, the cluster goes into a read-only state. In the read-only state, there are no create, read, update, or delete operations and no cluster changes. If the cluster experiences an unresponsive SSD, data loss is a risk. The Prism Central console offers an override option that allows users to continue writing to single-node clusters after it has entered read-only mode. Otherwise, be sure there is more than 1 SSD available for the system to place metadata. Refer to the `Prism Web Console Guide <https://portal.nutanix.com/#/page/docs/details?targetId=Web-Console-Guide-Prism-v56:wc-cluster-single-node-c.html>`_ for guidelines and requirements for Single-Node clusters.

- RF2 clusters should be sized to account for N+1 availability for BOTH compute AND storage. In the event of a node failure you should have enough resources to run all VMs and re-protect storage.

Replication Factor 3
....................

- RF3 clusters require a minimum of 5 nodes. While there are only three copies of the data, a fully healthy cluster requires 5 copies of metadata.

- RF3 clusters should ideally be sized to account for N+2 availablity for BOTH compute AND storage, as RF3 clusters are capable of sustaining up to 2 simultaneous node outages, but at a minimum should be sized for N+1 availability.

Videos
......

Refer to the following Nutanix TechTopX video for a detailed explanation of how Nutanix protects data:

.. raw:: html

  <iframe width="640" height="360" src="https://www.youtube.com/embed/OWhdo81yTpk?rel=0&amp;showinfo=0&amp;vq=hd1080" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe>

Refer to the following Nutanix TechTopX video for a detailed explanation of how Nutanix implements scalable metadata:

.. raw:: html

  <iframe width="640" height="360" src="https://www.youtube.com/embed/MlQczJhQI3U?rel=0&amp;showinfo=0&amp;vq=hd1080" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe>

Mixing Nodes
------------

- Nodes that use different CPU generations cannot be mixed in the same block. So a -G4 block must contain only -G4 nodes, a -G5 block must contain only -G5 nodes, and so on.

- Nodes that use different CPU generations can be mixed in the same cluster. No additional configuration is required in AOS or AHV, but 3rd party hypervisors require the configuration of CPU masking in order to support live migrations of VMs across nodes of different generations.

- Mixing nodes within the same cluster from multiple hardware vendors is not supported. However, you can manage separate clusters using the Prism/Prism Central web console regardless of the hardware/vendor type.

- Nodes that have RDMA enabled cannot be mixed in the same cluster with nodes that do not have RDMA enabled.

- Nodes that contain NVMe drives cannot be mixed in the same cluster with hybrid SSD/HDD nodes.

- Nodes that contain NVMe drives can be mixed in the same cluster with all-SSD nodes that do not contain NVMe drives.

- All-SSD nodes and hybrid SSD/HDD nodes can be mixed in the same cluster if the following conditions are met:
  - A minimum of two all-SSD nodes are required in mixed all-flash/hybrid clusters.
  - The minimum number of each node type (all-SSD/hybrid) must be equal to the cluster redundancy factor. For example, clusters with a redundancy factor of 2 must have a minimum of 2 hybrid and 2 all-SSD nodes.
  - Availability should also be considered when mixing all-SSD and hybrid nodes to ensure the minimum amount of SSD storage required for the workloads in the event of an all-SSD node failure (RF2) or failures (RF3).

- Nodes with significantly different amounts of storage capacity (e.g. NX-3060 and NX-6035 nodes) can be mixed in the same cluster if the following conditions are met:
  - The minimum number of each node type must be equal to the cluster redundancy factor. For example, clusters with a redundancy factor of 2 must have a minimum of 2 storage-heavy nodes.
  - Availability should also be considered when mixing nodes to ensure the minimum amount of total storage required for the workloads in the event of a storage-heavy node failure (RF2) or failures (RF3).
  - When mixing storage-light hybrid nodes with storage-heavy hybrid nodes, it is best practice to have an equal or greater amount of SSD capacity in each storage-heavy node as each storage-light node.

Data-at-Rest Encryption (DARE)
------------------------------

Nutanix supports DARE through Self-encrypting SSDs and HDDs, providing a FIPS 140-2 validated hardware-based solution. Use of self-encrypting drives (SEDs) also require the use of a third-party Key Management Service (KMS), such as Vormetric, SafeNet, and IBM SKLM.

Nutanix also supports DARE as a FIP 140-1 software-based solution, using Intel Advanced Encryption Standard instructions. Software-based encryption is available across AHV, ESXi, and Hyper-V on x86 platforms.

Refer to the `Information Security Tech Note <https://portal.nutanix.com/#/page/solutions/details?targetId=TN-2026_Information_Security:TN-2026_Information_Security>`_ for complete details on DARE.

Block Fault Tolerance
---------------------

Block fault tolerance is the Nutanix cluster's ability to make redundant copies of any data and place the data on nodes that are not in the same block. Metadata also must be block fault tolerant. A block is a rack-mountable enclosure that contains one to four Nutanix nodes. The power supplies, front control panels (ears), backplane, and fans are shared by all nodes in a block. With block fault tolerance enabled, guest VMs can continue to run with a block failure because redundant copies of guest VM data and metadata exist on other blocks.

Refer to the `NX and SX Series Hardware Administration Guide <https://portal.nutanix.com/#/page/docs/details?targetId=Hardware-Admin-Ref-AOS-v56:arc-block-awareness-c.html>`_ for complete details on Block Fault Tolerance requirements.

Erasure Coding
--------------

Unlike traditional data efficiency technologies like compression and deduplication, Erasure Coding (EC-X) does not depend on the content of that data, but rather whether or not the data is frequently updated/overwritten. This means that workloads that store lots of pre-compressed data (e.g. encoded audio, images, and video) or unique data that can't be deduplicated (e.g. logging data captured by a workload such as Splunk or ELK Stack) can still benefit from the space savings of Erasure Coding.

For details on minimum node requirements and sizing impact of Erasure Coding, refer to `Sizing assumptions for solutions with Erasure Coding (EC-X) <http://www.joshodgers.com/2015/10/07/sizing-assumptions-for-solutions-with-erasure-coding-ec-x/>`_ and `Erasure Coding Overheads <http://www.joshodgers.com/2016/02/17/erasure-coding-overheads-part-1/>`_.

Prism Central
-------------

Prism Central is used to monitor and manage entities across multiple Nutanix clusters. Prism Central is deployed as 1 or 3 VMs and is sized relative to the number of VMs managed across all clusters.

When sizing a cluster that will run Prism Central it is important to account for that overhead within Nutanix Sizer.

Refer to the latest `Prism Central Release Notes > Prism Central Scalability <https://portal.nutanix.com/#/page/docs/list?type=software>`_ for exact VM specification requirements.

CVM
---

The Controller Virtual Machine (CVM) is responsible for all cluster data services on each node. While Sizer accounts for CVM CPU and memory overhead appropriately for specific workloads, these values can be manually specified for Raw Input workloads.

Controller VM memory allocation requirements differ depending on the models and the features that are being used. Refer to the latest `Acropolis Advanced Administration Guide > Controller VM Memory Configurations <https://portal.nutanix.com/#/page/docs/list?type=software>`_ for exact VM specifications.

Support
-------

While product support doesn't directly impact sizing, ensuring the appropriate level of support to meet customer requirements is still a critical component of the overall solution. Nutanix offers multiple tiers of both software and hardware support, as well as programs for non-returnable disks, and on-site spare parts.

Click `here <https://www.nutanix.com/support-services/product-support/product-support-programs/>`_ for complete details on available Nutanix Support Programs.
