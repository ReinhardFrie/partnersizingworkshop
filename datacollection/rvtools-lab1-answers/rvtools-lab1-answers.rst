.. _rvtools_answers:

--------------------------------
Answers - Analyzing RVTools Data
--------------------------------

Summary
-------

**For Partner XChange Answers - will be posted towards end of lab**

Download the :download:`analyzed RVTools output file <SizingWorkshop-RVTools-Answers.xlsx>`.

============ =========== ================ ============ =============
Cluster Name Total vCPUs vCPU:pCore Ratio Total Memory Total Storage
============ =========== ================ ============ =============
Cluster1     257         4.0              1,195 GiB    18.7 TiB
Mgmt         80          4.0* (2.5)       233 GiB      1.7 TiB
DMZ          136         5.3              448 GiB      4.1 TiB
============ =========== ================ ============ =============

Notes
.....

We assumed that the analyzed clusters were already configured with N+1 availability, so we accounted for one less host worth of cores when calculating the total number of physical cores. We used this equation:

**(Number of Hosts - 1) * Average Total pCores per Host**

The current **DMZ** cluster is comprised of processors that are several generations old, so the single thread performance difference was factored into the vCPU:pCore ratio\:

**(Total vCPUs)/((Number of Hosts - 1) \* Average Total pCores per Host) \* ((Projected Processor Single Thread Score = 1952)/(Current Processor Single Thread Score = 1402))**

The **Mgmt** cluster calculated vCPU:pCore ratio is 2.5\:1, a possible indication that the current infrastructure has very low CPU utilization. In this instance we've elected to use an assumed value of 4:1. Any assumed values should be clearly communicated to the customer and corrborated by additional historical data if available.
