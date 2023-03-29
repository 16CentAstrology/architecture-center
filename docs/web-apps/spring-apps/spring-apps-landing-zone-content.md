
This reference architecture provides guidance for deploying Spring Boot applications in Azure Spring Apps. In this scenario, the workload is expected to use certain shared services provided by the organization as part of Azure landing zones. 

In this scenario, your organization wants to deploy the Spring Apps workload in an _Azure application landing zone_ that inherits the Corp. Management group. The workload is expected to integrate with preprovisioned shared resources in the _Azure platform landing zone_ that are managed by centralized teams.

> [!IMPORTANT]
> **What is an Azure landing zone?**
> An application landing zone is a Azure subscription in which the workload runs. It's connected to the organization's shared resources. Through that connection, it has access to basic infrastructure needed to run the workload, such as networking, identity access management, policies, and monitoring. The platform landing zones is a collection of various subscriptions, each with a specific function. For example, the Connectivity subscription provides centralized DNS resolution, on-premises connectivity, and network virtual appliances (NVAs) that's available for use by application teams. 
>
> As a workload owner, you benefit by offloading management of shared resources to central teams and focus on workload development efforts. The organization benefits by applying consistent governance and amortizing costs across multiple workloads.
> 
> We highly recommend that you understand the concept of [Azure landing zones](/azure/cloud-adoption-framework/ready/landing-zone/).


This article is part of the [**Azure Spring Apps landing zone accelerator**](/azure/cloud-adoption-framework/scenarios/app-platform/spring-apps/landing-zone-accelerator), which includes architectural guidance that cover the key technical design areas for this class of workload. This architecture makes design choices based on those recommendations. 

> [!TIP]
> ![GitHub logo](../../_images/github.svg) The architecture is backed by an [**example implementation**](https://github.com/Azure/azure-spring-apps-landing-zone-accelerator#azure-spring-apps-landing-zone-accelerator) that illustrates some of those choices. The implementation can be used as your first step towards production.


## Architecture

:::image type="content" source="./_images/spring-apps-reference-architecture-landing-zone.svg" alt-text="Diagram that shows an Azure Spring Apps workload in a landing zone." lightbox="./_images/spring-apps-reference-architecture-landing-zone.png":::

### Components

We've listed the components of this architecture by ownership to help determine your responsibilities and those that you share with the platform teams of the organization. For product documentation about Azure services, see [Related resources](#related-resources). 

#### Application team-owned resources

These resources are provisioned and owned by your team.

- **Azure Spring Apps Enterprise** hosts your Java Spring Boot applications in Azure. This tier is composed of the VMware Tanzu® Build Service™, Application Configuration Service for VMware Tanzu®, VMware Tanzu® Service Registry, Spring Cloud Gateway for VMware Tanzu®, and API portal for VMware Tanzu®. 
<<TBD: Why was this tier chosen?-- Provide some enterprise-y justifications.>>

- **Azure Application Gateway Standard_v2** is the load balancer that distributes incoming web traffic. This SKU has integrated Azure Web Application Firewall (WAF) that inspects traffic for Open Web Application Security Project (OWASP) vulnerabilities.

- **Azure Virtual Machine (VM)** acts as jump box for management operations. 

- **Azure Database for MySQL** stores application data. 
<<TBD: Why is the database in a separate scenario with the application? Also why is PostGres mentioned in the image?>>

**Azure Key Vault** stores secrets and configuration, such as connection string to the database. 

**Azure Log Analytics** is the monitoring sink that stores logs and metrics from the application and the Azure services.

**Azure Application Insights** is used as an Application Performance Management (APM) tool to collect all application monitoring data and store it directly within Log Analytics.

#### Platform team-owned resources

In this architecture these resources are assumed to be owned and maintained by the central teams of the organization. Your application depends on these services to reduce operational overhead and optimize cost.

**Azure Firewall**	is the NVA that inspects and restricts egress traffic.

**Azure Bastion** provides secure access tot he management jump box.

**Azure ExpressRoute**	Provides private connectivity from on-premises to Azure infrastructure.

**Azure DNS** provides cross-premises name resolution.

**VPN gateway**	connects the application with remote teams in your on-premises network.

## Deploy this scenario

A deployment for this reference architecture is available at [Azure Spring Apps Landing Zone Accelerator](https://github.com/Azure/azure-spring-apps-landing-zone-accelerator#azure-spring-apps-landing-zone-accelerator) on GitHub. The deployment uses Terraform templates. To deploy the architecture, follow the [step-by-step instructions](https://github.com/Azure/azure-spring-apps-landing-zone-accelerator/tree/main/Scenarios/ASA-Secure-Baseline/Terraform).


## Related resources

For product documentation on the Azure services used in this architecture, see these articles.

- [Azure Spring Apps Enterprise](/azure/spring-apps/overview#enterprise-plan)
- [Azure Application Gateway v2](/azure/application-gateway/overview-v2)
- [Azure Database for MySQL](/azure/mysql/overview)
- [Azure Key Vault](/azure/key-vault/)
- [Virtual Networks](/azure/virtual-network/)
- [Route tables](/azure/virtual-network/virtual-networks-udr-overview) 


## Next steps

Review the design areas of the [Azure Spring Apps landing zone accelerator](/azure/cloud-adoption-framework/scenarios/app-platform/spring-apps/landing-zone-accelerator).

## Related resources

- [Deploy Azure Spring Apps to multiple regions](spring-apps-multi-region.yml)
- [Expose Azure Spring Apps through a reverse proxy](spring-cloud-reverse-proxy.yml)

