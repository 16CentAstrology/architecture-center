The solution demonstrates how to use [Azure NetApp Files](/azure/azure-netapp-files/azure-netapp-files-introduction) as a storage solution for Siemens Teamcenter product lifecycle management (PLM). Doing so can help you  enhance performance and availability and improve data management and resource utilization of databases and shared file systems. Azure NetApp Files provides key data backup and redundancy features to improve the availability of your Teamcenter PLM data. 

## Architecture

This architecture illustrates a multi-zone deployment of Teamcenter PLM on Azure NetApp Files. The architecture is designed to deliver exceptional performance, scalability, and high availability (HA). By using Azure NetApp Files  volume placement and replication capabilities across multiple [Azure availability zones](/azure/reliability/availability-zones-overview#availability-zones), this architecture enables high performance, data resilience, and fault tolerance. Teamcenter PLM running on Azure NetApp Files enables you to distribute and replicate data across availability zones to create a resilient PLM solution in the cloud.

image Teamcenter PLM with Azure NetApp Files.

link 

**Availability zones:** The architecture uses two availability zones and replicates data across zones. This configuration helps to ensure the availability of data and to protect against data loss or corruption. The availability zones enable scalability. This flexibility enables you to meet evolving demands without sacrificing availability or performance.

**Resource tier:** Teamcenter PLM relies heavily on the resource tier to provide a scalable and reliable platform for managing product data and collaborating across teams and organizations. The resource tier provides access to resources like documents, 3D models, and drawings. This tier includes the primary and failover databases. The databases store database server data and log volumes. The resource tier also contains the Root FSC file server that stores digital assets (metadata) that are associated with the product data. It handles CAD files that are stored on data volumes. 

The architecture centralizes the digital assets and product data in the Root FSC servers and storage volumes in the same availability zone so that users can access the necessary data more quickly and easily. This tier also provides services for managing and maintaining these resources, like file versioning, revision control, and access permissions, which in many cases use Azure NetApp Files storage capabilities to function.

**Storage subnet:** The architecture deploys Azure NetApp Files volumes to the Storage subnet. Azure NetApp Files provides essential storage for Teamcenter PLM product data and digital assets. 

**Dataflow:** The databases and the Root FSC servers in the resource tier store and retrieve data from Azure NetApp Files volumes. The dataflow between the architecture tiers helps provide efficient and secure access to product data and digital assets because Azure NetApp Files volumes are hosted in the customer virtual network, in availability zones. Azure NetApp Files provides on-demand, non-disruptive scalability and HA across availability zones.

**Data replication:** The architecture uses SQL Server Always On availability groups to synchronously replicate the database across availability zones. Azure NetApp Files uses cross-zone replication to asynchronously replicate data across availability zones. 

## Considerations 

These considerations implement the pillars of the Azure Well-Architected Framework, which is a set of guiding tenets that you can use to improve the quality of a workload. For more information, see [Microsoft Azure Well-Architected Framework](/azure/architecture/framework).

### Reliability

Reliability ensures your application can meet the commitments you make to your customers. For more information, see [Overview of the reliability pillar](/azure/architecture/framework/resiliency/overview).

Azure NetApp Files provides JA with [built-in data replication, failover, and disaster recovery capabilities](/azure/azure-netapp-files/snapshots-introduction). These capabilities help ensure that your Teamcenter PLM database and CAD files are always available, even in the event of a regional, zonal, or software failure.

Azure NetApp Files provides a high [SLA](https://www.microsoft.com/licensing/docs/view/Service-Level-Agreements-SLA-for-Online-Services) for all tiers and all supported regions. Azure NetApp Files also supports provisioning volumes in [availability zones](/azure/azure-netapp-files/use-availability-zones) that you choose, and HA deployments across zones. 

Azure NetApp Files provides two data backup options: snapshots and backup. Snapshots provide a point-in-time backup of data. They enable you to quickly recover data in the event of data loss or corruption. Azure NetApp Files also has a fully managed backup solution. It provides long-term retention and recovery options for snapshots. You can combine snapshots and backups to get a comprehensive backup and recovery solution.

**Replicate data across availability zones.** Azure NetApp Files offers two important features: [availability zone volume placement](/azure/azure-netapp-files/use-availability-zones) and [cross-zone replication](/azure/azure-netapp-files/cross-zone-replication-introduction). Both contribute to HA architectures across availability zones for Teamcenter Root FSC and Oracle or SQL database servers.

We recommend that you use availability zone volume placement to ensure that the Teamcenter Oracle or SQL database servers, together with their associated data, are located in the same availability zone. This placement enhances performance by reducing network latency and provides fault isolation, enabling seamless fail over within the availability zone in case of failure when implemented together with Oracle Data Guard or SQL Server Always On availability groups.

You should use cross-zone replication when you require data redundancy and HA across multiple availability zones and don't want to use or require application-level replication. If you use cross-zone replication, any changes made in one availability zone are automatically replicated to volumes in other zones. This replication ensures data consistency and enables fail over to a secondary zone in case of a zone-level failure. It enhances data protection, disaster recovery, and overall system resilience for Teamcenter Root FSC file data.

**Configure Azure NetApp Files snapshots.** Snapshots provide a point-in-time backup of data, enabling quick recovery in the event of data loss or corruption. You can schedule snapshots to occur automatically on a regular basis or create them manually when you need to. You can create as many as 255 snapshots per volume with Azure NetApp Files.

You can set up Azure NetApp Files snapshots via schedule policy or manually in the Azure portal or by using Azure SDKs or APIs. Your application consistency requirements can help you determine which method to use:

- For applications like Oracle or SQL Server that require application consistency, create snapshots manually. Because SQL Server and Oracle databases require application consistency when storage-based snapshots are used, you need to ensure that correct application-to-storage snapshot orchestration occurs. You can achieve this orchestration by implementing an application freeze-snapshot-thaw cycle that enables snapshots to be taken consistently without interupting database operation. For more information about snapshot consistency with Azure NetApp Files, see [SQL Server snapshot consistency](https://techcommunity.microsoft.com/t5/azure-architecture-blog/managing-sql-server-2022-t-sql-snapshot-backup-with-azure-netapp/ba-p/3654798) or [Oracle snapshot consistency](/azure/azure-netapp-files/azacsnap-introduction).
- Use a snapshot policy for Root FSC file data. The Root FSC file data hosted on Azure NetApp Files volumes doesn't require any application consistency or orchestration because those volumes are general file shares. We recommend that you use an [Azure NetApp Files snapshot policy](/azure/azure-netapp-files/snapshots-manage-policy) to automatically schedule and initiate snapshots at required intervals.

**Configure Azure NetApp Files backup.** [Azure NetApp Files backup](/azure/azure-netapp-files/backup-introduction) expands the data protection capabilities of Azure NetApp Files by providing a fully managed backup solution for long-term recovery, archive, and compliance. The service stores backups in Azure storage. These backups are independent from volume snapshots that are available for near-term recovery or cloning. You can restore backups taken by this service to new Azure NetApp Files volumes within the region. Azure NetApp Files backup supports both policy-based (scheduled) backups and manual (on-demand) backups.  

**Use Azure NetApp Files file/folder and volume restore options.** Azure NetApp Files provides file/folder granular restore and volume restore options. Both options are useful for Teamcenter PLM data recovery. (See the following diagram.)

You can use **file/folder granular restore** to recover specific files or folders from a snapshot or backup. This option is useful when only certain data needs to be restored because it saves time and storage space. You can also use the Azure NetApp Files snapshot feature to recover data. By using snapshots, you can recover data from previous points in time without needing to restore it from a backup. This feature can help you quickly restore individual files have been accidentally deleted or corrupted.

You can use **volume restore** to recover an entire volume from a snapshot or backup. This option is useful when an entire volume is lost or corrupted and a complete restore is necessary (for example, if a Ransomware attack occurs). This feature can help you quickly restore an entire volume of Teamcenter PLM data, including CAD files, application and database servers, and other critical data.

- **Take regular snapshots.** If you take regular snapshots of data, you can restore the system to a previous state in a matter of minutes, minimizing any potential downtime and ensuring that critical data is always available when you need it. 
 
  There are three major areas in a Teamcenter PLM architecture where advanced data protection and recoverability are important and where snapshots ensure fast protection and recovery: 

  - Individual file restores from file shares
  - Database data consistency backups
  - Volume restores to recover from virus and Ransomware attacks and database problems
  
The following diagram shows Azure NetApp Files snapshot restore options that are specific to Teamcenter PLM application areas.

:::image type="content" source="media/restore-options.png" alt-text="Diagram that shows Azure NetApp Files snapshot restore options that are specific to Teamcenter PLM application areas." lightbox="media/restore-options.png":::

Use online snapshots for most restore operations, rather than using offline backups (vaulted snapshots). For more information, see [How Azure NetApp Files snapshots work](/azure/azure-netapp-files/snapshots-introduction).

**Set up cross-region replication for disaster recovery.** For higher availability, you can replicate the storage volumes to another Azure region by using Azure NetApp Files [cross-region replication](/azure/azure-netapp-files/cross-region-replication-introduction). (See the following image.) There are two main advantages to replicating the storage volumes by using Azure NetApp Files replication rather than application-level or host-level replication: 

- There's no additional load on the application virtual machines or your virtual network. Azure NetApp Files replicates the storage content without using any compute infrastructure resources.
-  It eliminates the need to continuously run virtual machines in the destination region during normal operations. The destination virtual machines don't need to be running to support this scenario.

The typical recovery point objective (RPO) for this solution is less than 20 minutes when the cross-region replication update interval is set to 10 minutes. The recovery time objective (RTO), or the maximum tolerable business application downtime, depends on how long it takes to bring up the application and provide access to the data at the second site. The storage portion of the RTO, for breaking the peering relationship to activate the destination volume and provide read and write data access to the second site, is expected to be complete within a minute. For more information, see [Create volume replication for Azure NetApp Files](/azure/azure-netapp-files/cross-region-replication-create-peering).

This architecture shows a disaster recovery solution that uses cross-region replication:

:::image type="content" source="media/cross-region-replication.png" alt-text="Diagram that shows a disaster recovery solution that uses cross-region replication." lightbox="media/cross-region-replication.png" border="false":::
 
#### Dataflow 

1.	The client tier, web app tier, enterprise tier, and resource tier in the production environment use the Azure NetApp Files in the storage tier. 
2.	The applications in their respective tiers use application replication *if needed* to replicate data to an Azure disaster recovery (DR) region. We recommend that you use Azure NetApp Files cross-region replication if you can.
3.	Azure NetApp Files cross-region replication asynchronously replicates the data volumes, including all snapshots, to a DR region to facilitate fail over if there's a regional disaster. To save money when you're not performing a DR fail over, you can use Standard service levels for the destination volumes during replication.
4.	In case of a disaster, the client, web app, enterprise and resource tiers in the *DR environment* use the Azure NetApp Files storage tier for data storage and performance in the DR environment. In a DR scenario, you should change the Azure NetApp Files volume in the DR environment to the Premium or Ultra tier required for running the applications.
5.	When the disaster is over, you can initiate a failback event to return the replicated data to its original region. The failback event replicates changes made to the data back to the original region in stages, including any new snapshots created on the DR site, with critical data prioritized first. After replication is complete, applications can resume normal operations in the original region.

**Configure database HA.**  You can combine SQL Server Always On availability groups or Oracle Data Guard with the Azure NetApp Files availability zone volume placement feature to build HA architectures for these applications.

- **Use SQL Server Always On availability groups with Azure NetApp Files.** SQL Server Always On availability groups enable synchronous or asynchronous replication of SQL Server databases across availability zones. This replication ensures data redundancy and failover capabilities in the SQL Server ecosystem. Configure Azure NetApp Files to use availability zone volume placement. Availability zone placement locates the underlying storage for SQL Server databases in the same availability zone as the SQL Server instances.
- **Use Oracle Data Guard with Azure NetApp Files.** Configure Oracle Data Guard to replicate Oracle databases between availability zones for data redundancy and HA. Use Azure NetApp Files with availability zone volume placement to ensure that the storage that supports the Oracle databases is located in the same availability zone as the Oracle database instances.

### Security 

Security provides assurances against deliberate attacks and the abuse of your valuable data and systems. For more information, see [Overview of the security pillar](/azure/architecture/framework/security/overview).

Azure NetApp Files provides security features like [encryption](/azure/azure-netapp-files/faq-security#can-the-storage-be-encrypted-at-rest) and access control. These features help protect Teamcenter PLM databases from unauthorized access, data breaches, and cyber threats. Azure NetApp Files provides a level of [security](/azure/azure-netapp-files/faq-security#can-the-network-traffic-between-the-azure-vm-and-the-storage-be-encrypted) because data remains in your virtual networks. There's no endpoint that can be accessed publicly. All [data is encrypted at rest](/azure/azure-netapp-files/faq-security#can-the-storage-be-encrypted-at-rest) at all times. 

**Use enhanced file access security.** You can use [SMB share permissions](/azure/azure-netapp-files/faq-smb#can-the-smb-share-permissions-be-changed) and [NTFS access control lists (ACLs)](/azure/azure-netapp-files/azure-netapp-files-create-volumes-smb#ntfs-file-and-folder-permissions), in addition to [NFS export policies](/azure/azure-netapp-files/azure-netapp-files-configure-export-policy) and [UNIX permissions and ownership mode](/azure/azure-netapp-files/configure-unix-permissions-change-ownership-mode) or [NFSv4.1 ACLs](/azure/azure-netapp-files/configure-access-control-lists) to enhance file access security.

**Use network security groups.** Apply [network security groups](/azure/virtual-network/network-security-groups-overview) on Azure virtual networks and Azure NetApp Files delegated subnets (configured with [standard network features](/azure/azure-netapp-files/azure-netapp-files-network-topologies#configurable-network-features)).

**Consider encrypting data in transit.** Optionally, [enable encrypt data-in-transit](/azure/azure-netapp-files/faq-security#can-the-network-traffic-between-the-azure-vm-and-the-storage-be-encrypted) on any volume exported via NFSv4.1, SMB, or dual-protocol. 

**Use Azure Policy.** [Azure Policy](/azure/governance/policy/overview) can help you enforce organizational standards and assess compliance at scale. Azure NetApp Files supports Azure Policy via [custom and built-in policy definitions](/azure/azure-netapp-files/azure-policy-definitions).

### Cost optimization

Cost optimization is about reducing unnecessary expenses and improving operational efficiency. For more information, see [Overview of the cost optimization pillar](/azure/architecture/framework/cost/overview). 

Azure NetApp Files provides a [cost-effective storage solution with a pay-as-you-go model](/azure/azure-netapp-files/azure-netapp-files-cost-model#calculation-of-capacity-consumption). You pay only for the storage and performance resources that you use, and there's no upfront investment in hardware or infrastructure.

**Understand the Azure NetApp Files cost model.** Understanding the [cost model for Azure NetApp Files](/azure/azure-netapp-files/azure-netapp-files-cost-model) can help you manage your expenses. Azure NetApp Files billing is based on provisioned storage capacity, which you allocate by creating capacity pools. Capacity pools are billed monthly based on a set cost per allocated GiB per hour.

**Consider dynamically resizing volumes and capacity pools**. If your capacity pool size requirements fluctuate (for example, because of variable capacity or performance needs), consider [dynamically resizing your volumes and capacity pools](/azure/azure-netapp-files/azure-netapp-files-resize-capacity-pools-or-volumes) to balance cost with your capacity and performance needs.

**Consider dynamically changing the volume tier (service level).** If your capacity pool size requirements remain the same but performance requirements fluctuate, consider [dynamically changing the service level of a volume](/azure/azure-netapp-files/dynamic-change-volume-service-level). You can provision and deprovision capacity pools of different types throughout the month, providing just-in-time performance and reducing costs during periods when you don’t need high performance. It is recommended to use these capabilities when setting up a DR scenario across Azure regions, where the secondary region gets provisioned on the lowest, most cost-effective (standard) tier, and volumes only move volumes to the appropriate performance in case of a disaster/failover event.

**Use cross-region and cross-zone replication.** The use of [cross-region and cross-zone replication](/azure/azure-netapp-files/cross-zone-replication-introduction) takes away the need for host-based replication mechanisms, avoiding virtual machine and software license cost.

Azure NetApp Files also optimizes cost for the [Oracle](https://techcommunity.microsoft.com/t5/azure-architecture-blog/run-your-most-demanding-oracle-workloads-in-azure-without/ba-p/3264545) and [SQL Server](/azure/azure-netapp-files/solutions-benefits-azure-netapp-files-sql-server) database applications, by allowing heavy, storage I/O intensive databases to run on smaller Azure VM SKUs within a series. You should leverage [Azure constrained vCPU SKUs](/azure/virtual-machines/constrained-vcpu) where possible to save on compute and software license cost.

**Understand performance requirements.** Based on your capacity and performance requirements, determine the Azure NetApp Files service level you need (Standard, Premium, or Ultra). Then use the [Azure Pricing calculator](https://azure.microsoft.com/pricing/calculator) to evaluate the costs for these components:

- Siemens Teamcenter PLM on Azure components
- Database on Azure components
- Azure NetApp Files, cross-region replication, and backup
- Managed disk (OS boot disks)
- Networking components

**Use the Azure NetApp Files performance calculator.** The [Azure NetApp Files performance calculator](https://anftechteam.github.io/calc/advanced/) helps you determine the correct Azure NetApp Files storage tier for your cost and performance needs.

**Consult with an Azure Solutions Architect.** It’s recommended to consult an Azure Cloud Solution Architect (CSA) to assist with application sizing and selecting the smallest applicable VM SKU.

### Operational excellence

Operational excellence covers the operations processes that deploy an application and keep it running in production. For more information, see [Overview of the operational excellence pillar](/azure/architecture/framework/devops/overview). Azure NetApp Files for Siemens Teamcenter PLM can help you comply with several of the Azure Well-architected Framework Operational excellence topics:

**Use native monitoring features.** Azure NetApp Files provides [built-in monitoring and diagnostics capabilities](/azure/azure-netapp-files/monitor-azure-netapp-files) that enable you to monitor the performance and health of your Teamcenter PLM application, CAD file shares, and databases. You can set up alerts and notifications for critical events, such as file system capacity and performance issues, and take corrective actions proactively.

**Manage performance.** Azure NetApp Files offers non-disruptive, on-demand [capacity scaling](/azure/azure-netapp-files/azure-netapp-files-resize-capacity-pools-or-volumes) and [service level changes](/azure/azure-netapp-files/dynamic-change-volume-service-level), allowing you to scale up or down quickly as required by your Teamcenter PLM applications, CAD file shares, and databases. This, together with volume resizing or manual QoS settings, helps to [manage the performance](/azure/azure-netapp-files/azure-netapp-files-performance-considerations) and availability of your application.

**Automate infrastructure deployments.** Azure NetApp Files enables you to automate the deployment of your Teamcenter PLM application infrastructure, CAD file shares, and databases. You can use [Azure Resource Manager templates](/azure/azure-netapp-files/azure-netapp-files-sdk-cli#azure-resource-manager-templates) to define and deploy the required resources, such as file shares, storage accounts, virtual machines, and databases, to support your application.

**Test deployments.** Azure NetApp Files provides a reliable and scalable platform for deploying your Teamcenter PLM application code, CAD files, and databases. You can use the [cloning via snapshot restore feature](/azure/azure-netapp-files/snapshots-introduction#restoring-cloning-an-online-snapshot-to-a-new-volume) to create a new volume for testing purposes quickly. This helps to ensure that your application is always up-to-date and stable, and you can test new releases or upgrades quickly and easily.

**Test environments.** Azure NetApp Files allows you to create and manage test environments quickly and easily by [cloning your production data via snapshot restore to a new volume](/azure/azure-netapp-files/snapshots-introduction#restoring-cloning-an-online-snapshot-to-a-new-volume). This helps you to isolate testing activities from production environments and provides a quick and easy way to test disaster recovery scenarios. You can also use the same process to test new releases or upgrades in a non-disruptive manner. This helps to ensure that your application is robust and reliable.

### Performance efficiency

Performance efficiency is the ability of your workload to scale to meet the demands placed on it by users in an efficient manner. For more information, see [Performance efficiency pillar overview](/azure/architecture/framework/scalability/overview). Azure NetApp Files provides high-performance storage with low latency, high throughput, and consistent performance. This means that your Teamcenter PLM [database will be able to handle high volumes of data, users, and transactions without any performance degradation](https://techcommunity.microsoft.com/t5/azure-architecture-blog/run-your-most-demanding-oracle-workloads-in-azure-without/ba-p/3264545). It also means that your CAD files will be able to be accessed and modified quickly and efficiently, which can help improve productivity and collaboration.

**Pick the right compute and performance tiers.** To ensure optimal performance of your Teamcenter PLM environment, use the right Azure VM types and Azure NetApp Files performance tier. Azure NetApp Files volumes are available in three performance tiers (or ‘service level’): [Ultra, Premium, and Standard](/azure/azure-netapp-files/azure-netapp-files-service-levels#supported-service-levels). Choose the tier that best suits your performance requirements, taking into account that available performance bandwidth [scales with the size of a volume](/azure/azure-netapp-files/azure-netapp-files-service-levels#throughput-limits). You can optimize performance by resizing or changing the service level of a volume at any time. For more information about the Azure NetApp Files cost model, see these [pricing examples](/azure/azure-netapp-files/azure-netapp-files-cost-model#pricing-examples).  

The VM type and performance tiers need to meet storage needs of the (1) Siemens Teamcenter servers (Root FSC), (2) database servers, and (3) engineering workstations (*see Figure 2*).

image 

Figure 2: Varied storage tier requirements across infrastructure components.

- *Siemens Teamcenter Servers (Root FSC)* require medium-to-high storage performance for file data management. Depending on volume capacities, the Azure NetApp Files Premium tier is typically recommended unless [performance sizing](https://anftechteam.github.io/calc/advanced) shows otherwise. You should use Azure D or F series VMs for the Root FSC servers. Azure NetApp Files provides high level SLAs to maintain operability and IOPS levels.
- *Database servers ([SQL Server](/azure/azure-netapp-files/solutions-benefits-azure-netapp-files-sql-server) or [Oracle](https://techcommunity.microsoft.com/t5/azure-architecture-blog/run-your-most-demanding-oracle-workloads-in-azure-without/ba-p/3264545))* require minimal latency between it and storage. Azure NetApp Files provides latencies of less than 1 millisecond. You should use E/M-series Azure VMs for your database servers. Where possible, enable [constrained vCPU](/azure/virtual-machines/constrained-vcpu) to save on SKU and license cost. For the most demanding database applications, you should use the Azure NetApp Files Ultra tier unless database [performance sizing](https://anftechteam.github.io/calc/advanced) shows otherwise.
- *Engineering workstations* should use NV series VMs for the engineering workstations and leverage NVIDIA GPUs and NVIDIA GRID technology for desktop accelerated applications and virtual desktops. Azure NetApp Files also optimizes GPU enabled engineering workstations by providing fast and concurrent access to file data to multiple workstations and therefore fostering collaboration. Typically, Azure NetApp Files Standard tier is sufficient for this use case, especially since the standard tier provides same low latency performance experience as the other tiers even at low bandwidth requirements – also in situations with high concurrency (like many users access the volume in parallel).

**Scale your storage capacity.** With Azure NetApp Files, you can easily scale your storage capacity and performance to meet your changing business needs. You can [add or remove capacity and performance resources on-demand, without any downtime or data migration which helps improve TCO](/azure/azure-netapp-files/azure-netapp-files-cost-model#pricing-examples).

Azure NetApp Files provides several features that help to scale on-demand for performance and cost, while ensuring that the changes are transparent to the applications running on top:

1.	[Dynamic service level changes](/azure/azure-netapp-files/dynamic-change-volume-service-level): Azure NetApp Files allows you to change the service level of your volumes on-demand, without any disruption to your applications. This means that you can increase or decrease the performance or capacity of your volumes as required, without impacting the availability of your applications. This helps you to optimize the performance and cost of your storage resources based on your workload requirements.
2.	[Adjusting performance when using ‘auto QoS’](/azure/azure-netapp-files/azure-netapp-files-service-levels#throughput-limit-examples-of-volumes-in-an-auto-qos-capacity-pool): By default, Azure NetApp Files automatically applies Quality of Service (QoS) to your volumes to ensure that they receive the required level of performance. When you resize your volumes, the QoS settings are automatically adjusted to reassign the level of performance. This means that your applications continue to receive the adjusted level of performance as the storage capacity changes.
3.	[Adjusting performance when using 'Manual QoS'](/azure/azure-netapp-files/azure-netapp-files-service-levels#throughput-limit-examples-of-volumes-in-a-manual-qos-capacity-pool): Azure NetApp Files also allows you to manually assign QoS policies to your volumes to ensure that they receive the required level of performance. Without resizing your volumes, you can adjust the QoS settings to optimize the performance and cost of your storage resources. This means that you can fine-tune the performance of your applications based on your workload requirements.

All of these changes are transparent to the Siemens Teamcenter components running on top of Azure NetApp Files. The applications continue to access the volumes in the same way, and the performance and availability are maintained at the required level. This helps you to scale your storage resources on-demand for performance and cost, without disrupting your business operations.

**Run performance test.** Customer deployments and performance validation tests of Azure NetApp Files for Siemens Teamcenter PLM have validated significant decreases in execution times and significant improvements in both read and write times. This results in overall productivity gains and optimizes resource utilization. Use the [Azure NetApp Files Performance Calculator](https://anftechteam.github.io/calc/) to get started with an initial sizing and cost estimation. 

### Potential use cases

**Requirements engineering, model-based systems engineering, and PLM process management:** Azure NetApp Files offers high-performance storage with low latency, enabling faster access to critical data and improving overall system performance. This is particularly important for use cases such as requirements engineering, model-based systems engineering, and PLM process management, which rely heavily on accessing and processing large volumes of data.

**Bill of materials management and product configuration:** Azure NetApp Files also provides flexible storage options that can scale up or down based on changing business needs. This is particularly useful for use cases such as bill of materials management and product configurations, which require significant storage capacity and may experience frequent changes in data volume.

**Product cost management:** Azure NetApp Files provides advanced data protection features, including point-in-time backups and disaster recovery options, which can help ensure data availability and minimize the risk of data loss. This is important for use cases such as product cost management, where accurate and up-to-date data is critical for making informed decisions.

**Development across departments and domains:** Azure NetApp Files contributes to improving development across departments and domains by providing a high-performance, reliable, and scalable storage service for Siemens Teamcenter PLM deployments. With Azure NetApp Files, PLM data can be accessed quickly and easily, allowing for efficient collaboration between different departments and domains.

**Mechanical design management:** Azure NetApp Files can provide high-speed access to large design files, allowing engineers to quickly iterate on designs and make necessary changes. For electronics and electrical CAD management, Azure NetApp Files can provide a centralized location for storing and managing CAD data, making it easy for engineers to collaborate on designs and share data across departments.

**Software design management:** Azure NetApp Files can provide high-performance storage for build artifacts and development environments, enabling fast build and deployment times. 

**Product document management:** Azure NetApp Files can provide a centralized location for storing and managing product documentation, making it easy to access and share information across different teams.

**Product visualization and desktop mockup:** Azure NetApp Files can provide high-speed access to large graphic files and simulations, enabling faster visualization and mockup times. And for simulation process and data management, Azure NetApp Files can provide fast and reliable storage for simulation data, allowing engineers to quickly run simulations and analyze results.

**Product visibility and insight:** Azure NetApp Files enables the delivery of product visibility and insight by providing fast and reliable access to PLM data and analytics, which can help organizations make informed decisions and optimize their product development processes. For example, PLM analytics can provide insights into the performance of products and help identify areas for improvement, while sustainable product development can help organizations track the environmental impact of their products and make more sustainable choices. 

## Contributors

*This article is maintained by Microsoft. It was originally written by the following contributors.*

Principal authors: 

- Tilman Schroeder - Cloud Lead Automotive, NetApp
- Geert van Teylingen - Azure NetApp Files Group Product Manager


Other contributors:
 
- Sunita Phanse – Senior Technical Program Manager, Microsoft

line 

## Next steps

To find out more about Azure capabilities related to implementing a robust data management layer for PLM, read the following: 

- Azure Marketplace solutions for Siemens Teamcenter
- Oracle solutions on Azure NetApp Files
- SQL Server solutions on Azure NetApp Files
- Benefits of using Azure NetApp Files for SQL Server Deployment
- Azure Virtual Desktop solutions with Azure NetApp Files
- File sharing solutions with Azure NetApp Files
- Azure Architecture Center solutions with Azure NetApp Files
- Azure NetApp Files Performance Calculator
- Azure Pricing calculator
