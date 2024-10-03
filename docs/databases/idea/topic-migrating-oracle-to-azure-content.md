# Migrating Oracle database workloads to Azure

## Introduction to migrating Oracle database workloads to Azure

This article describes considerations and recommendations when migrating Oracle database workloads from on-premises to Azure. The article assumes that you are familiar with Oracle Database technologies and Azure networking. The following scenarios are covered in the article:

- Physical migration of Oracle databases to Azure virtual machines
- Physical migration of Oracle databases to Oracle Database@Azure, Exadata Database Service (OD@A)

## Scenario

Consider the following initial scenario: 

- You have one or more Oracle databases running in your on-premises data center, which you are looking to migrate to Azure.
- The databases are running on Oracle Database 19c, Enterprise Edition.
- The databases are RAC enabled and for disaster recovery they are replicated via Oracle Data Guard to another data center geographically distant from where the primary is located.
- You need to migrate the databases to Azure with the minimum amount of downtime.
- Application services dependent on the database will be migrated to Azure as well.
- You have established network connectivity to Azure through ExpressRoute, and use a hub and spoke network topology in Azure.
- In the hub VNet the traffic has to traverse a third-party NVA (FortiGate, CheckPoint, Cisco or other), which doubles as a routing device, ensuring that traffic to/from Azure undergoes traffic inspection, while at the same time being fully routable.

![Diagram1](_images/oracle-database-migration-to-azure.jpg)

## Implementation checklist

- If you will be migrating to Azure virtual machines, review [Migrating Oracle database workloads to Azure virtual machines](migrating-oracle-to-azure-iaas.yml).
- If you will be migrating to Oracle Database@Azure, Exadata Database Service, review [Migrating Oracle database workloads to Oracle Database@Azure, Exadata Database Service](migrating-oracle-to-odaa-exadata.yml).

## Next steps
TBD