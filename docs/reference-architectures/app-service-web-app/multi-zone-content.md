<!-- cSpell:ignore CNAME -->



This reference architecture shows how to run an Azure App Service application in a zone-redundant configuration to achieve high availability. Zone-redundant services replicate your services and data across Availability Zones to protect from single points of failure.

Multi-zone architectures are a good alternative to [multi-region architectures], offering less complexity and lower cost, while still meeting the availability and recovery requirements for most customers.

![Reference architecture for a web application with high availability](./images/multi-zone-web-app-diagram.png)

<!-- *See a [full working sample of this architecture in Azure Samples], including Visio file, Bicep template and Bill of materials.* -->

## Architecture

This architecture builds on the additional availability provided by [Availability Zones infrastructure] found in many Azure regions today. For a list of Azure regions that support Availability Zones see [Azure regions with Availability Zones][az-regions].

Availability Zones spread a solution across multiple zones within a region, allowing for an application to continue functioning when one zone fails. Most foundational and mainstream Azure services, as well as many Specialized Azure services provide support for Availability Zones today. All of the Azure services in this architecture are [Zone Redundant], simplifying deployment and management. For a list of Azure services that support Availability Zones see [Azure Services that support Availability Zones][az-services].

This is essentially an active/active design within a single region, where high availability is provided as a feature of a platform. In most cases failures are automatically mitigated without user intervention.

### Front Door

Azure Front Door is a global, scalable entry-point that uses the Microsoft global edge network to create fast, secure, and widely scalable web applications. Front Door is resilient to failures, including failures to an entire Azure region.

In this architecture [Azure Front Door] serves as an edge for all public HTTP traffic ingressing into the Azure Region. Front Door provides CDN style cache, DDOS, WAF functionality, and WAN acceleration in one convenient service. Alternatively, [Azure CDN] and [Azure Application Gateway] could be used instead.

## Recommendations

Your requirements might differ from the architecture described here. Use the recommendations in this section as a starting point.

### App Services & Functions

[App Service Premium v2, Premium v3](app-services-zr), [Isolated v3](ise-zr) and [Azure Functions Elastic Premium SKUs](functions-zr) can be enabled for zone-redundancy. In this configuration App Service Plan instances are distributed across multiple availability zones to protect from data-center or zone failure. A zone-redundancy enabled App Service plan, Isolated App Service Plan, or Elastic Premium plan should be deployed with at least 3 instances To achieve zone redundancy. 

Function Apps should either be hosted in a dedicated App Service Plan (alongside other apps) with zone-redundancy enabled, or as Premium Functions in a zone-redundant Elastic Premium plan.

### SQL Database

Use an Azure SQL DB Tier that supports Availability Zones, for example [Premium and Business Critical service tier zone redundant availability][sql-azs] can be enabled with no downtime and at no additional cost (to a locally redundant Premium and Business Critical service). By selecting a zone redundant configuration, you can make your Premium or Business Critical databases resilient to a much larger set of failures, including catastrophic datacenter outages, without any changes to the application logic.

Azure SQL Database Business Critical or Premium tiers configured as Zone Redundant Deployments have an availability guarantee of at least 99.995%. In the event of a single zone failure, zone redundancy provides full data durability with RPO=0 and availability with RTO=0.

### Cosmos DB

Enable Zone-redundancy in Azure Cosmos DB when selecting a region to associate with your Azure Cosmos account. With Availability Zone (AZ) support, Azure Cosmos DB will ensure replicas are placed across multiple zones within a given region to provide high availability and resiliency to zonal failures. Availability Zones provide a 99.995% availability SLA with no changes to latency. In the event of a single zone failure, zone redundancy provides full data durability with RPO=0 and availability with RTO=0.

### Storage

For Azure Storage, use [Zone-redundant storage][zrs] (ZRS). With ZRS storage, Azure replicates your data synchronously across three Azure availability zones in the region. ZRS offers durability for Azure Storage data objects of at least 99.9999999999% (12 9's) over a given year.

A write request to a storage account that is using ZRS happens synchronously. The write operation returns successfully only after the data is written to all replicas across the three availability zones. With ZRS, your data is still accessible for both read and write operations even if a zone becomes unavailable.

### Service Bus

The [Service Bus Premium SKU supports Availability Zones][servicebus-az], providing fault-isolated locations within the same Azure region. Service Bus manages three copies of messaging store (1 primary and 2 secondary). Service Bus keeps all the three copies in sync for data and management operations. If the primary copy fails, one of the secondary copies is promoted to primary with no perceived downtime.

### Cache for Redis

Cache for Redis supports [zone redundant configurations][redis-zr] in the Premium and Enterprise tiers. A zone redundant cache can place its nodes across different Azure Availability Zones in the same region. It eliminates datacenter or AZ outage as a single point of failure and increases the overall availability of your cache.

### Cognitive Search

You can utilize [Availability Zones with Azure Cognitive Search][cog-search-az] by adding two or more replicas to your search service. Each replica will be placed in a different Availability Zone within the region. If you have more replicas than Availability Zones, the replicas will be distributed across Availability Zones as evenly as possible.

### Key Vault

Key Vault is zone redundant in any region where Availbility zones are available. There is no additional configuration required.


<!-- links -->

[guidance-web-apps-scalability]: ./scalable-web-app.yml
[guidance-web-apps-scalability-devops]: ./scalable-web-app.yml#devops-considerations
[zrs]: https://docs.microsoft.com/en-us/azure/storage/common/storage-redundancy#zone-redundant-storage
[services-by-region]: https://azure.microsoft.com/regions/#services
[sql-rpo]: /azure/sql-database/sql-database-business-continuity#sql-database-features-that-you-can-use-to-provide-business-continuity
[sql-azs]:https://docs.microsoft.com/en-us/azure/azure-sql/database/high-availability-sla#premium-and-business-critical-service-tier-zone-redundant-availability
[az-regions]:https://docs.microsoft.com/en-us/azure/availability-zones/az-region#azure-regions-with-availability-zones
[az-services]:https://docs.microsoft.com/en-us/azure/availability-zones/az-region
[servicebus-az]:https://docs.microsoft.com/en-us/azure/service-bus-messaging/service-bus-outages-disasters#availability-zones
[redis-zr]:https://docs.microsoft.com/en-us/azure/azure-cache-for-redis/cache-high-availability#zone-redundancy
[cog-search-az]:https://docs.microsoft.com/en-us/azure/search/search-performance-optimization#availability-zones
[app-services-zr]:https://docs.microsoft.com/en-us/azure/app-service/how-to-zone-redundancy
[functions-zr]:https://docs.microsoft.com/en-us/azure/azure-functions/azure-functions-az-redundancy
[ise-zr]:https://docs.microsoft.com/en-us/azure/app-service/environment/overview-zone-redundancy