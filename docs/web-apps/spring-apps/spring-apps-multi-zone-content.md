This reference architecture describes the approach for running application workloads on Azure Spring Apps. The design is focused on achieving high availability through zonal redundancy. Application services and data are automatically replicated across in multiple zones within a region. The intent is to prevent the application from going down if all datacenters in a zone experience outage. 

> [!TIP]
> ![GitHub logo](../../_images/github.svg) The architecture is backed by an [**example implementation**](https://github.com/Azure-Samples/azure-spring-apps-multi-zone) that illustrates some of design choices. Consider the implementation as your first step towards production.

## Architecture

:::image type="content" source="./_images/zone-redundant-spring-apps-reference-architecture.png" alt-text="Diagram that shows a multi-region Azure Spring Apps reference architecture." lightbox="./_images/zone-redundant-spring-apps-reference-architecture.png":::

*Download a [Visio file](https://arch-center.azureedge.net/ha-zone-redundant-spring-apps-reference-architecture.vsdx) that contains this architecture diagram.*

### Components

Here are the Azure services used in this architecture. For product documentation about Azure services, see [Related resources](#related-resources). 

- **Azure Spring Apps Standard** hosts a sample Java Spring Boot application implemented as microservices. 

- **Azure Application Gateway Standard_v2** is the load balancer and manages traffic to the applications. It acts a local reverse proxy in a region where your application runs. 

    This SKU has integrated Azure Web Application Firewall (WAF) that provides centralized protection of your web applications from common exploits and vulnerabilities. Web Application Firewall on the Application Gateway tracks OWASP exploits.

- **Azure DNS** resolves requests sent to the host name of the application to the public endpoint of Azure Application Gateway.

- **Azure Database for MySQL** stores state in a backend relational database. 

- **Azure Key Vault** stores application secrets and the certificates that Application Gateway and Azure Spring Apps use.


### Workflow

1. The user accesses the application by using the HTTP host name of the application; for example, `www.contoso.com`. Azure DNS resolves the request for this host name to the public endpoint of Azure Application Gateway.

1. Application Gateway with integrated WAF inspects the request and forwards the allowed traffic to the IP address of the load balancer in the provisioned Azure Spring Apps instance.

1. The internal load balancer routes the traffic to the backend services.

1. As part of processing the request, the application communicates with other Azure services inside the virtual network. For example, it reaches Key Vault for secrets and the database for storing state. 

## Redundancy

Building redundancy in the workload will minimize single points of failure. In this architecture, architecture components are replicated across zones within the selected region. Azure services aren't supported in all regions and and not all regions support zones. Before selecting a region, check regional and zone support in [Products available by region](https://azure.microsoft.com/global-infrastructure/services/).

Azure Spring Apps supports zonal redundancy. When you do so, all underlying infrastructure of the service is spread across multiple availability zones. This zone spreading supports an overall higher availability of your applications that use the service. Java Spring Boot applications are horizontally scaled without any code changes. 

The Application Gateway is also set up to use multiple availability zones.

Azure Database for MySQL with the flexible server deployment option was chosen to support high availability with automatic failover. You can choose between `Zone-redundant HA` and `same-zone HA`. That choice depends on your latency requirements. With high availability configuration, flexible server automatically provisions and manages a standby replica. If there's an outage, committed data isn't lost. 


## Network security

[Azure Virtual Network](https://azure.microsoft.com/products/virtual-network) is the fundamental building block for a private network in Azure. This architecture uses a virtual network for each region that you use for deployment. Creating subnets will create isolation. Azure Spring Apps requires a dedicated subnet for the service runtime and a separate subnet for Spring Boot applications.

The design incorporates several PaaS services that participate in processing a user request. Strict network controls are placed to prevent unauthorized public access from the internet.

##### Private connectivity

You can control access to those services from public internet by using private endpoints.  These network interfaces use private IP addresses to bring the services into the virtual networks. 

The architecture has Azure services that automatically set up the private endpoints. 

Azure Spring Apps is deployed into that network using [vnet-injection](/azure/spring-apps/how-to-deploy-in-azure-virtual-network). As part of provisioning, a subnet and the neccessary private endpoints are created. The application is isolated from the internet, systems in private networks, other Azure services, and even the service runtime. The application is accessed by reaching the private IP address.

The database also follows a similar model. The [flexible server deployment mode of Azure Database for MySQL](https://azure.microsoft.com/products/mysql) supports virtual network integration through a dedicated subnet.

There are other services, such as Azure Key Vault, which are connected to the virtual network through [Azure Private Link](https://azure.microsoft.com/products/private-link). For Private Link, you need to enable a private endpoint so that public network access is disabled. For more information about private endpoints for Azure Key Vault, see [Integrate Key Vault with Azure Private Link](/azure/key-vault/general/private-link-service?tabs=cli).

Private endpoints should be put in a dedicated subnet. Private IP addresses to the private endpoints are assigned from that subnet.

The private endpoint and network-integrated connections use an [Azure private DNS zone](/azure/dns/private-dns-getstarted-cli).

##### Controls on the inbound traffic



## Scalability


## Monitoring


## Deployment



- [Azure Private Link](https://azure.microsoft.com/products/private-link) provides private endpoints that connect privately and securely to services. These network interfaces use private IP addresses to bring the services into the virtual networks. This solution uses private endpoints for the key vault.
- [Managed identities](/azure/active-directory/managed-identities-azure-resources/overview) in [Azure Active Directory (Azure AD)](https://azure.microsoft.com/products/active-directory) provide automatically managed identities that applications can use to connect to resources that support Azure AD authentication. Applications can use managed identities to get Azure AD tokens without having to manage any credentials. This architecture uses managed identities for several interactions, for example between Azure Spring Apps and the key vault.

, which allow incoming calls only from Application Gateway.

1. The components inside the virtual networks use [private endpoints](/azure/private-link/private-endpoint-overview) to connect privately and securely to other Azure services. This solution uses private endpoints to connect to Azure Key Vault. [Azure Key Vault](/azure/key-vault/general/overview) stores application secrets and certificates. The microservices that run in Azure Spring Apps use the application secrets. Azure Spring Apps, Application Gateway, and Azure Front Door use the certificates for host name preservation.

1. 

### Alternatives

The following sections discuss alternatives for several aspects of this architecture.

#### Multi-region deployment

To increase application resilience and reliability, you can alternatively deploy the application to multiple regions. If you do, add an additional [Azure Front Door](/azure/frontdoor/front-door-overview) or [Azure Traffic Manager](/azure/traffic-manager/traffic-manager-overview) service to load balance requests to your applications across regions.

However, multi-region deployment doubles the cost of your setup, because you duplicate the full setup to a secondary region. For this reason, the choice is often made to provide an active-passive setup, where only one region is active and deployed. In this case, you add a global load balancer to the multi-region setup to provide an easy way of failing over your workloads after a secondary region becomes active. Whether active-active or active-passive is the best choice for your workload depends on the availability requirements you have for your application.

The biggest challenge with a multi-region setup is replicating the data for your application between multiple regions. This isn't an issue with the multi-zone setup. Azure availability zones are connected by a high-performance network with a round-trip latency of less than 2 ms. This latency is OK for most applications.

You can also combine a multi-zone solution with a multi-region solution.

#### Back-end database

This architecture uses a MySQL database . You can also use other database technologies, like [Azure SQL Database](/azure/azure-sql/azure-sql-iaas-vs-paas-what-is-overview), [Azure Database for PostgreSQL](/azure/postgresql/single-server/overview), [Azure Database for MariaDB](/azure/mariadb/overview), or [Azure Cosmos DB](/azure/cosmos-db/introduction).

You can also connect some of these databases to your virtual network through [Azure Private Link](https://azure.microsoft.com/products/private-link). Azure Private Link isn't necessary for [the flexible server deployment mode of Azure Database for MySQL](https://azure.microsoft.com/products/mysql), which directly supports virtual network integration through a dedicated subnet.

#### Reverse proxy setup

The current solution uses Application Gateway as a reverse proxy. You can, however, use different reverse proxies in front of Azure Spring Apps. You can combine Azure Application Gateway with Azure Front Door, or you can use Azure Front Door instead of Azure Application Gateway.

For information about different reverse proxy scenarios, how to set them up, and their security considerations, see [Expose Azure Spring Apps through a reverse proxy](spring-cloud-reverse-proxy.yml).

#### Key vault

This solution stores the application secrets and certificates in a single key vault. However, because application secrets and the certificates for host name preservation are different concerns, you might want to store them in separate key vaults. This alternative adds another key vault to your architecture.

## Solution details

This architecture describes a multi-zone design for Azure Spring Apps. This architecture is useful when you want to:

- Increase the availability of your applications.
- Increase the overall resilience and service level objective (SLO) of your application.

### Potential use cases

- Public website hosting
- Intranet portal
- Mobile app hosting
- E-commerce
- Media streaming
- Machine learning workloads

> [!IMPORTANT]
> For business-critical workloads, we recommend combining zone redundancy and regional redundancy to achieve maximum reliability and availability, with zone-redundant services deployed across multiple Azure regions.
> For more information, refer to the [Global distribution](/azure/architecture/framework/mission-critical/mission-critical-application-design#global-distribution) section of the mission-critical design methodology, and the [Mission-critical baseline architecture](/azure/architecture/reference-architectures/containers/aks-mission-critical/mission-critical-intro).
> You can also use the [Deploy Azure Spring Apps to multiple regions guidance](/azure/architecture/reference-architectures/microservices/spring-apps-multi-region) for an automated setup across regions.

## Recommendations

The following recommendations apply to most scenarios. Follow these recommendations unless you have specific requirements that override them.

### Front-end IP Addresses

[Availability zone](/azure/virtual-network/ip-services/public-ip-addresses#availability-zone) support is available for public IP addresses with a standard SKU. When you use availability zones in your architecture,  make sure you use availability zones for all the components in your setup, including the public IP address used by the Application Gateway.

### Application Gateway

[Application Gateway v2](/azure/application-gateway/overview-v2) can be spread across multiple availability zones. When you use availability zones in your architecture, make sure you use availability zones for all the components in your setup, including the Application Gateway.

Enable [Web Application Firewall](/azure/web-application-firewall/) on your OWASP-enabled Application Gateway.

### Key Vault

Key Vault is automatically zone redundant in any region where availability zones are available. The instance of Key Vault that this architecture uses is deployed so that back-end services can access secrets. 



### Automated deployment

Automate your deployments as much as possible. You should automate infrastructure deployment and application code deployments.

Automating infrastructure deployments guarantees that infrastructure is configured identically, avoiding configuration drift (for example, between environments). Infrastructure automation can also help you test failing over and quickly bringing up a secondary region.

You can also use a [blue-green](/azure/architecture/example-scenario/blue-green-spring/blue-green-spring) or canary deployment strategy.

## Considerations

These considerations implement the pillars of the Azure Well-Architected Framework, which is a set of guiding tenets that can be used to improve the quality of a workload. For more information, see [Microsoft Azure Well-Architected Framework](/azure/architecture/framework).

### Reliability

Reliability ensures your application can meet the commitments you make to your customers. For more information, see [Overview of the reliability pillar](/azure/architecture/framework/resiliency/overview).

This architecture explicitly increases the availability of your application over a single-zone deployment.

### Security

Security provides assurances against deliberate attacks and the abuse of your valuable data and systems. For more information, see [Overview of the security pillar](/azure/architecture/framework/security/overview).

From a networking perspective, this architecture is locked down to allow incoming calls only through the public endpoint exposed by Application Gateway. From the Application Gateway, the calls route to the back-end Azure Spring Apps service. Communication from Azure Spring Apps to supporting services, like the back-end database and the key vault, is also locked down by using either private endpoints or network integration.

This architecture provides extra security by using a managed identity to connect between different components. For example, Azure Spring Apps uses a managed identity to connect to Key Vault. Key Vault allows Azure Spring Apps only minimal access to read the needed secrets, certificates, and keys.

You should also protect your virtual networks with [Azure DDoS Protection](/azure/ddos-protection/ddos-protection-overview). DDoS Protection, combined with application design best practices, provides enhanced mitigations to defend against distributed denial-of-service (DDoS) attacks.

### Cost optimization

Cost optimization is about looking at ways to reduce unnecessary expenses and improve operational efficiencies. For more information, see [Overview of the cost optimization pillar](/azure/architecture/framework/cost/overview).

This solution has a higher cost than a single-zone version. This higher cost is because some components in the setup are deployed in multiple zones. Instead of one instance, they run two or even three instances. However, [Azure Spring Apps](/azure/spring-apps/how-to-enable-redundancy-and-disaster-recovery?tabs=azure-cli#pricing) doesn't have any extra cost associated with it when you enable zone redundancy on the service.

To address costs:

- You can deploy different applications and application types to a single instance of Azure Spring Apps. By deploying multiple applications, the cost of the underlying infrastructure is shared across applications.
- Azure Spring Apps supports application autoscaling triggered by metrics or schedules, which can improve utilization and cost efficiency.
- You can use Application Insights in [Azure Monitor](/azure/azure-monitor/overview) to lower operational costs. A comprehensive logging solution provides visibility for automation to scale components in real time. Analyzing log data can also reveal inefficiencies in application code that you can address to improve costs and performance.
- If you use Front Door in an alternative setup instead of Application Gateway, you're charged on a per-request basis. With Front Door, you don't have to provision multiple Application Gateway instances, and cost is calculated per actual request to your application.

All the services this architecture describes are pre-configured in an [Azure pricing calculator estimate](https://azure.com/e/414c5e0b15494e5081cc9f008d82fdaa) with reasonable default values for a small-scale application. You can update this estimate based on the throughput values you expect for your application.

### Operational excellence

Operational excellence covers the operations processes that deploy an application and keep it running in production. For more information, see [Overview of the operational excellence pillar](/azure/architecture/framework/devops/overview).

This architecture addresses the same aspects of operational excellence as the [single-region reference architecture for Azure Spring Apps](/azure/spring-cloud/reference-architecture). You can automate the full architecture rollout.

For operational excellence, integrate all components of this solution with Azure Monitor Logs to provide end-to-end insight into your application.

### Performance efficiency

Performance efficiency is the ability of your workload to scale to meet the demands placed on it by users in an efficient manner. For more information, see [Performance efficiency pillar overview](/azure/architecture/framework/scalability/overview).

This multi-zone architecture is better suited than a single-zone deployment to meet application demands because it spreads the load across availability zones.

Depending on your database setup, you might incur extra latency when data needs to be synchronized between zones.

This architecture has several components that can autoscale based on metrics:

- Application Gateway supports autoscaling. For more information, see [Scale Application Gateway v2 and WAF v2](/azure/application-gateway/application-gateway-autoscaling-zone-redundant).
- Azure Spring Apps also supports autoscaling. For more information, see [Set up autoscale for applications](/azure/spring-apps/how-to-setup-autoscale).

## Deploy this scenario

A deployment for this reference architecture is available at [Azure Spring Apps multi zone reference architecture](https://github.com/Azure-Samples/azure-spring-apps-multi-zone) on GitHub. The deployment uses Terraform templates. To deploy the architecture, follow the [step-by-step instructions](https://github.com/Azure-Samples/azure-spring-apps-multi-zone#getting-started).

## Contributors

*This article is maintained by Microsoft. It was originally written by the following contributor.*

Principal author:

- [Gitte Vermeiren](https://www.linkedin.com/in/gitte-vermeiren-b1b2221) | FastTrack for Azure Engineer

*To see non-public LinkedIn profiles, sign in to LinkedIn.*

## Next steps

To increase application resilience and reliability, you can alternatively deploy the application to multiple regions.

> [!div class="nextstepaction"]
> [Deploy Azure Spring Apps to multiple regions](spring-apps-multi-region.yml)

To integrate this workload with shared services managed by central teams in your organization, deploy it in Azure application landing zone. 

> [!div class="nextstepaction"]
> [Azure Spring Apps integrated with landing zones](spring-apps-landing-zone.yml)

## Related resources

For documentation on the Azure services and features used in this architecture, see these articles.

- [Azure Spring Apps](https://azure.microsoft.com/products/spring-apps)
- [Azure Application Gateway v2](/azure/application-gateway/overview-v2)
- [Azure Database for MySQL](/azure/mysql/overview)
- [Azure Key Vault](/azure/key-vault/)
- [Azure DNS](https://azure.microsoft.com/products/dns)
- [Azure Web Application Firewall](https://azure.microsoft.com/products/web-application-firewall)
- [Azure Private Link](https://azure.microsoft.com/products/private-link)
- [Managed identities](/azure/active-directory/managed-identities-azure-resources/overview)

We recommend these guides to get deeper understanding around the choices made in this architecture:

- [Expose Azure Spring Apps through a reverse proxy](spring-cloud-reverse-proxy.yml)
- [High-availability blue/green deployment](../../example-scenario/blue-green-spring/blue-green-spring.yml)




-  provides centralized protection of your web applications from common exploits and vulnerabilities. Web Application Firewall on the Application Gateway tracks OWASP exploits.
-  


- [Azure Virtual Network](https://azure.microsoft.com/products/virtual-network) is the fundamental building block for a private network in Azure. This solution uses a virtual network for each region that you use for deployment.
- [Azure Private Link](https://azure.microsoft.com/products/private-link) provides private endpoints that connect privately and securely to services. These network interfaces use private IP addresses to bring the services into the virtual networks. This solution uses private endpoints for the key vault.
- [Managed identities](/azure/active-directory/managed-identities-azure-resources/overview) in [Azure Active Directory (Azure AD)](https://azure.microsoft.com/products/active-directory) provide automatically managed identities that applications can use to connect to resources that support Azure AD authentication. Applications can use managed identities to get Azure AD tokens without having to manage any credentials. This architecture uses managed identities for several interactions, for example between Azure Spring Apps and the key vault.