---
tags:
  - aws
share: "true"
---


# AWS Application Discovery Service



AWS Application Discovery Service helps you plan your migration to the AWS cloud by collecting usage and configuration data about your on-premises servers and databases.

## Buzzwords

* **Application Discovery Service** is integrated with **AWS Migration Hub** and **AWS Database Migration Service Fleet Advisor**. 

* **Migration Hub** aggregates your migration status information into a single console.
* You can view the discovered servers, group them into applications, and then track the migration status of each application from the Migration Hub console in your home Region. You can use DMS Fleet Advisor to assess migrations options for database workloads.
* you can export data about the network connections that exist between servers.

* All discovered data is stored in your **AWS Migration Hub home Region**. Therefore, you **must set your home Region in the Migration Hub**
* data can be exported for analysis in  Amazon Athena and Amazon QuickSight
* Agentless discovery can be performed by deploying the Application Discovery Service Agentless Collector (Agentless Collector) (OVA file) through your VMware vCenter. After Agentless Collector is configured, it identifies virtual machines (VMs) and hosts associated with vCenter. Agentless Collector collects the following static configuration data: Server hostnames, IP addresses, MAC addresses, disk resource allocations, database engine versions, and database schemas. Additionally, it collects the utilization data for each VM and database providing the average and peak utilization for metrics such as CPU, RAM, and Disk I/O.

* Agent-based discovery can be performed by deploying the AWS Application Discovery Agent on each of your VMs and physical servers. The agent installer is available for Windows and Linux operating systems. It collects static configuration data, detailed time-series system-performance information, inbound and outbound network connections, and processes that are running.

    The term host refers to either a physical server or a VM.

    Collected data is in measurements of kilobytes (KB) unless stated otherwise.

    Equivalent data in the Migration Hub console is reported in megabytes (MB).

    The polling period is in intervals of approximately 15 seconds and is sent to AWS every 15 minutes.

    Data fields denoted with an asterisk (*) are only available in the .csv files that are produced from the agent's API export function.


* **Agentless Collector** currently supports data collection from VMware VMs and from database and analytics servers. Future modules will support collection from additional virtualization platforms, and operating system level collection.
  
* **VMWare Discovery** use the Agentless Collector to collect system information without having to install an agent on each VM.Agentless Collector captures system performance information and resource utilization for each VM running in the vCenter, regardless of what operating system is in use. However, it cannot “look inside” each of the VMs, and as such, cannot figure out what processes are running on each VM nor what network connections exist. Therefore, if you need this level of detail and want to take a closer look at some of your existing VMs in order to assist in planning your migration, you can install the Discovery Agent on an as-needed basis.

* **Database Discover** The Agentless Collector database and analytics data collection module captures metadata and performance metrics that provide insight into your data infrastructure. The database and analytics data collection module uses LDAP in Microsoft Active Directory to gather information about the OS, database, and analytics servers in your network. Then, the data collection module periodically runs queries to collect actual utilization metrics of CPU, memory, and disk capacity for the databases and analytics servers


To use Application Discovery Service, the following is assumed:

    You have signed up for AWS. For more information, see Setting up Application Discovery Service.

    You have selected a Migration Hub home Region. For more information, see the documentation regarding home Regions.

Here's what to expect:

    The Migration Hub home Region is the only Region where Application Discovery Service stores your discovery and planning data.

    Discovery agents, connectors, and imports can be used in your selected Migration Hub home Region only.

    For a list of AWS Regions where you can use Application Discovery Service, see the Amazon Web Services General Reference.




