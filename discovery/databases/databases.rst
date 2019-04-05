.. _databases:

---------
Databases
---------

Overview
--------

Nutanix offers many advantages for running Tier 0 and Tier 1 databases, including the ability to start small and grow linearly, have predictable performance, colocate transactional and analytical workloads, support physical database workloads, and enable simplified copy data management with the upcoming Era launch.

Common databases run on Nutanix include traditional relational databases such as Microsoft SQL Server, Oracle, PostgreSQL, and MySQL, as well as NoSQL databases such as MongoDB and Apache Cassandra.

Basic Discovery
----------------

- **Source\\Target** - Is the database currently baremetal? Is it going to be virtualized on Nutanix? These questions will determine if we are sizing for the compute aspect of the database or if we are only exposing volume groups via iSCSI (Acropolis Block Services) to the database and sizing for storage capacity/performance.

- **Number of Database VMs** - Also helpful to understand how many instances per database VM. Nutanix recommends a single instance per VM, allowing for scale out utilization of cluster resources.

- **VM Specifications** - vCPUs, Memory, Number of Virtual Disks, Size of each Virtual Disk.

- **Working Set** - How large is the database and what is the change rate? Production databases should be sized for 100% of the database to live in SSD.

  - **SQL Server** - `Xtract for DB <https://portal.nutanix.com/#/page/xtract>`_ can target SQL server to determine working set size
  - **Oracle** - Ask your prospect to provide an Oracle Automatic Workload Repository (AWR) report. Requires Oracle Diagnostics Pack licensing.

- **High Availability** - What approach is the database using for HA/DR?

  - **SQL Server** - AlwaysOn Availability Groups (AAG), Failover Cluster Instance (FCI)
  - **Oracle** - Real Application Clusters (RAC), ASM Cluster File System (ACFS)

- **Current IOPS** - Any historical data the customer can provide on the current performance (IOPS/latency). May not directly impact the sizing of the cluster, but can be beneficial in positioning the proposed solution.

- **Licensing** - What level of licensing does the prospect have for their database platform? This could impact alternate HA solutions, data collection for sizing, etc.

- **Type of Database**

  - **OLTP**
  - **OLAP**
  - **DSS**

Additional Resources
--------------------

`SQL Server 2016 Best Practices Guide <https://portal.nutanix.com/#/page/solutions/details?targetId=BP-2015-Microsoft-SQL-Server:BP-2015-Microsoft-SQL-Server>`_

`MySQL Best Practices Guide <https://portal.nutanix.com/#/page/solutions/details?targetId=BP-2056-MySQL-on-Nutanix:BP-2056-MySQL-on-Nutanix>`_

`PostgreSQL Best Practices Guide <https://portal.nutanix.com/#/page/solutions/details?targetId=BP-2061-PostgreSQL-on-Nutanix:BP-2061-PostgreSQL-on-Nutanix>`_

`Virtualizing MongoDB Best Practices Guide <https://portal.nutanix.com/#/page/solutions/details?targetId=BP-2023_MongoDB_on_Nutanix:BP-2023_MongoDB_on_Nutanix>`_

`Oracle on AHV Best Practices Guide <https://portal.nutanix.com/#/page/solutions/details?targetId=BP-2073-Oracle-on-AHV:BP-2073-Oracle-on-AHV>`_

`Oracle PeopleSoft Campus Solutions on Nutanix AHV Reference Architecture <https://portal.nutanix.com/#/page/solutions/details?targetId=RA-2103-PeopleSoft-Campus-Solutions:RA-2103-PeopleSoft-Campus-Solutions>`_

`Oracle RAC on an Extended Cluster with Nutanix ABS Reference Architecture <https://portal.nutanix.com/#/page/solutions/details?targetId=RA-2011-Oracle-RAC-Extended-Cluster-ABS:RA-2011-Oracle-RAC-Extended-Cluster-ABS>`_

`Informatica PowerCenter Grid on Nutanix Reference Architecture <https://portal.nutanix.com/#/page/solutions/details?targetId=RA-2012-Informatica-PowerCenter-Grid:RA-2012-Informatica-PowerCenter-Grid>`_

`Virtualizing Oracle Databases Best Practices Guide <https://portal.nutanix.com/#/page/solutions/details?targetId=BP-2000-Oracle-on-Nutanix:BP-2000-Oracle-on-Nutanix>`_

Refer to `Nutanix Portal Solution Documentation <https://portal.nutanix.com/#/page/solutions>`_ for additional Tech Note and Solution Note documents.
