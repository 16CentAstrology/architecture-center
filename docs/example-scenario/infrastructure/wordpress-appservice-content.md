<!-- cSpell:ignore wordpress -->

Use [Azure Front Door](/azure/frontdoor/front-door-overview), [Azure App Service](/azure/app-service/quickstart-wordpress) and other Azure services to deploy a highly scalable and secure installation of WordPress.

## Architecture

[![Architecture overview of the WordPress deployment in App Service](media/wordpress-appservice.png)](media/wordpress-appservice.png#lightbox)

> [!NOTE]
> This architecture can be extended and combined with other tips and recommendations that are not specific to any particular WordPress hosting method. [Learn more about tips for WordPress](/azure/architecture/example-scenario/infrastructure/wordpress)

### Dataflow

This scenario covers a scalable and secure installation of WordPress that uses Ubuntu web servers and MariaDB. There are two distinct data flows in this scenario the first is users access the website:

1. Users access the front-end website through a CDN (Azure Front Door *or* Azure CDN).
2. The CDN load balances requests across Azure App Service instances that WordPress is running on and pulls any data that isn't cached from the WordPress web app 
3. The Azure load balancer distributes ingress traffic to App Service instances.
4. The WordPress application pulls any dynamic information out of the managed [Azure Database for MySQL - Flexible Server](https://learn.microsoft.com/en-us/azure/mysql/flexible-server/overview), access privately via Private Endpoint.
5. The WordPress application pulls any dynamic information out of the Maria DB clusters via Private Endpoint, all static content is hosted in [Azure Blob Storage](/azure/storage/blobs/storage-blobs-overview).

### Components

- [Azure Front Door](https://azure.microsoft.com/products/frontdoor) *or* [Azure Content Delivery Network (CDN)](https://azure.microsoft.com/products/cdn) is a Microsoft’s modern cloud Content Delivery Network (CDN), distributed network of servers that efficiently delivers web content to users. CDNs minimize latency by storing cached content on edge servers in point-of-presence locations near to end users.
- [Virtual networks](https://azure.microsoft.com/products/virtual-network) allow resources such as VMs to securely communicate with each other, the Internet, and on-premises networks. Virtual networks provide isolation and segmentation, filter and route traffic, and allow connection between locations. The two networks are connected via Vnet peering.
- [Azure DDoS Protection Standard](/azure/ddos-protection/ddos-protection-overview), combined with application-design best practices, provides enhanced DDoS mitigation features to provide more defense against DDoS attacks. You should enable [Azure DDOS Protection Standard](/azure/ddos-protection/ddos-protection-overview) on any perimeter virtual network.
- [Network security groups](/azure/virtual-network/security-overview) contain a list of security rules that allow or deny inbound or outbound network traffic based on source or destination IP address, port, and protocol. The virtual networks in this scenario are secured with network security group rules that restrict the flow of traffic between the application components.
- [Azure Key Vault](https://azure.microsoft.com/products/active-directory) is used to store and tightly control access to passwords, certificates, and keys.
- [Azure Database for MySQL - Flexible server](https://azure.microsoft.com/products/mysql/) is database used to store WordPress data.

## Scenario details

This solution is ideal for small to medium-sized WordPress installations, as it provides the scalability, reliability, and security of the Azure platform without the need for complex configuration or management. For larger or storage-intensive installations, see other [hosting options for WordPress](/azure/wordpress#wordpress-hosting-options-on-azure).

### Potential use cases

Other relevant use cases include:

- Media events that cause traffic surges.
- Blogs that use WordPress as their content management system.
- Business or e-commerce websites that use WordPress.
- Web sites built using other content management systems.

### Alternatives

- [Azure Cache for Redis](https://azure.microsoft.com/products/cache/) can be used to host key-value cache for WordPress performance optimization plugins, shared between all App Service instances.

## Considerations

These considerations implement the pillars of the Azure Well-Architected Framework, which is a set of guiding tenets that can be used to improve the quality of a workload. For more information, see [Microsoft Azure Well-Architected Framework](/azure/architecture/framework).

### Availability

The App Service comes with load balancing and health check features, if there's instance failure.

The CDN (Front Door) is global service and supports origins deployed across multiple regions (App Service deployed in another regions). In addition, caching all responses on the CDN level can provide a small availability benefit when the origin isn't responding. However, it's important to note that caching shouldn't be considered a complete availability solution.

The Azure Blob Storage storage can be replicated between paired regions. For more information, see [Azure Storage redundancy](/azure/storage/common/storage-disaster-recovery-guidance).

For high availability of Azure Database for MySQL, see [High availability concepts in Azure Database for MySQL - Flexible Server](/azure/mysql/flexible-server/concepts-high-availability).

### Scalability

This scenario hosts front-end in App Service. With autoscale feature, the number of instances that run the front-end application tier can automatically scale in response to customer demand, or based on a defined schedule. For more information, see [Get started with autoscale in Azure](/azure/azure-monitor/autoscale/autoscale-get-started).

For more resiliency and scalability guidance, see the [resiliency checklist](/azure/architecture/checklist/resiliency-per-service) in the Azure Architecture Center.

### Security

Security provides assurances against deliberate attacks and the abuse of your valuable data and systems. For more information, see [Overview of the security pillar](/azure/architecture/framework/security/overview).

All the virtual network traffic into the front-end application tier and protected by [WAF on Azure Front Door](/azure/web-application-firewall/afds/afds-overview). No outbound Internet traffic is allowed from the database tier. No access to private storage is allowed from public. For more information about WordPress security, see [General WordPress security&performance tips](/azure/wordpress#general-wordpress-securityperformance-tips).

For general guidance on designing secure scenarios, see the [Azure Security Documentation][security].

### Resiliency

This scenario supports use of multiple regions, data replication and auto-scalling. These networking components distribute traffic to the pods, and include health probes that ensure traffic is only distributed to healthy instances. All of these networking components are fronted via a CDN. This approach makes the networking resources and application resilient to issues that would otherwise disrupt traffic and affect end-user access.

For general guidance on designing resilient scenarios, see [Designing reliable Azure applications](/azure/architecture/framework/resiliency/app-design).

### Cost optimization

Cost optimization is about looking at ways to reduce unnecessary expenses and improve operational efficiencies. For more information, see [Overview of the cost optimization pillar](/azure/architecture/framework/cost/overview).

There are a couple main things to consider:

- How much traffic are you expecting in terms of GB/month? The amount of traffic has the biggest effect on your cost, as it determines the number of App Service instances. Additionally, it directly correlates with the amount of data that is surfaced via the CDN.
- What is the expected amount of hosted data? It's important to consider this since Azure Storage pricing is based on used capacity.
- How much new data are you going to be writing to your website? New data written to your website correlates with how much data is mirrored across the regions.
- How much of your content is dynamic? How much is static? The variance around dynamic and static content influences how much data has to be retrieved from the database tier versus how much is cached in the CDN.

## Contributors

*This article is maintained by Microsoft. It was originally written by the following contributors.*

Principal author:

[Vaclav Jirovsky](https://www.linkedin.com/in/vaclavjirovsky) | Cloud Solution Architect

Other contributors:

- Adrian Calinescu | Sr. Cloud Solution Architect

## Next steps

Product documentation:

- [What is Azure Front Door?](/azure/frontdoor/front-door-overview)
- [What is Azure Web Application Firewall?](/azure/web-application-firewall/overview)
- [What is Azure Blob Storage?](/azure/storage/blobs/storage-blobs-overview)
- [What is Azure Virtual Network?](/azure/virtual-network/virtual-networks-overview)
- [About Azure Key Vault](/azure/key-vault/general/overview)
- [Quickstart: Create a WordPress site](/azure/app-service/quickstart-wordpress)

Microsoft Learn modules:

- [Load balance your web service traffic with Front Door](/training/modules/create-first-azure-front-door/)
- [Implement Azure Key Vault](/training/modules/implement-azure-key-vault)
- [Introduction to Azure Virtual Networks](/training/modules/introduction-to-azure-virtual-networks)

## Related resources

- [Ten design principles for Azure applications](../../guide/design-principles/index.md)
- [Scalable cloud applications and site reliability engineering](../../example-scenario/apps/scalable-apps-performance-modeling-site-reliability.yml)

<!-- links -->

[security]: /azure/security