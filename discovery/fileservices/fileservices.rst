-------------
File Services
-------------

Overview
--------

Acropolis File Services offers native, scale out file services on AHV and ESXi with support for SMB 2.0, 2.1, 3.0 (basic protocol support without specific SMB 3.0 features) and NFSv4 protocols.

Basic Discovery
---------------

- **Protocol/Client** - Which network protocol do you use to connect to file shares/exports? Which client operating systems are you using to connect to file shares/exports? Refer to the latest `AFS Release Notes <https://portal.nutanix.com/#/page/docs/list?type=software>`_ for a current list of supported protocols and clients.

- **Share Type** - Understanding the share type determines the additional data required to properly size an AFS deployment, as well as how data is sharded across File Server VMs (FSVMs).

  - **Home**

    - **Total Users**
    - **Total Storage**
    - **Working Set**
  - **Department**

    - **Total Users**
    - **Total Storage**
    - **Working Set**
    - **Total Shares**
  - **Application Data**

    - **Sequential Throughput**
    - **Read/Write Percentage**
    - **Total Storage**
    - **Working Set**
    - **Total Shares**

- **Data Protection** - By default AFS will perform snapshots of the file server with a 1 hour RPO, retaining the last 24 hourly, 7 daily, 4 weekly, and 3 monthly snapshots. Do you require replication via native Async replication or DFS replication with PeerSync to a secondary site? What is the expected change rate?

- **Antivirus** - Which antivirus solution are you using for your current filer? AFS supports ICAP scanning and is supported with multiple vendors. Refer to the latest `AFS Release Notes <https://portal.nutanix.com/#/page/docs/list?type=software>`_ for a current list of supported antivirus solutions.

- **Directory Services** What are you using to authenticate access to file shares/exports? Refer to the latest `AFS Release Notes <https://portal.nutanix.com/#/page/docs/list?type=software>`_ for a current list of supported directory services.

- **System Limits** - Refer to the latest `AFS Release Notes <https://portal.nutanix.com/#/page/docs/list?type=software>`_ for a current list of system limits, including:
  - Number of Connections per FSVM
  - Max throughput per node
  - Number of FSVMs per AFS cluster
  - Max RAM per FSVM
  - Max vCPUs per FSVM
  - Max size per Home share
  - Max size per General Purpose share
  - Max number of characters per file/share/cluster name
  - Max RPO for Async DR
