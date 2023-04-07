---
title: Test
description: Test
author: BryanLa
ms.author: bryanla
ms.date: 04/07/2023
ms.topic: conceptual
ms.service: architecture-center
ms.subservice: well-architected
products:
  - azure
---


# [BCDR](#tab/bcdr)
### Guidance for VMs

##### Availability Zones

Availability zones are unique physical locations within an Azure region. Each zone is made up of one or more datacenters with independent power, cooling, and networking. The physical separation of availability zones within a region limits the impact to applications and data from zone failures

By replicating VMs across availability zones, you can protect your applications and data from a zone failure. This is how Azure meets the industry-best [VM uptime service-level agreement (SLA)](https://azure.microsoft.com/support/legal/sla/virtual-machines/v1_9). For more information, see [Building solutions for high availability using availability zones](./building-solutions-for-high-availability.yml).

##### Update domains

Spreading resources across availability zones also protects an application from planned maintenance. When VMs are distributed across three availability zones, they are, in effect, spread across three update domains. The Azure platform recognizes this distribution across update domains to ensure that VMs in different zones aren't updated at the same time.

### Guidance for PaaS and other components

With zone-redundant services, the distribution of the workload is a feature of the service and is handled by Azure. Azure automatically replicates the resource across zones without requiring your intervention. For example, zone-redundant load balancer, Azure Application Gateway, virtual private network (VPN), zone-redundant storage.

Using Application Gateway or a Standard Load Balancer configured as zone-redundant, traffic can be routed to VMs located across zones with a single IP address, which will survive zone failures. The load frontend IP can be used to reach all (non-impacted) VMs no matter the zone. One or more availability zones can fail and the data path survives as long as one zone in the region remains healthy.

# [Compute](#tab/compute)

In this baseline architecture we are using Virtual Machine Scale Sets (VMSS) with Flexible orchestration to facilitate the operation of virtual machines at cloud scale. Unlike uniform VMSS, with flexible orchestration Azure enables you to allocate and manage VMs individually. You have full control over the virtual machine lifecycle, as well as network interfaces and disks using the standard Azure VM APIs and commands. At the same time, by joining VM to a flexible VMSS, you get an orchestration layer that facilitates achieving high availability at scale with identical or multiple virtual machine types. Flexible orchestration offers high availability guarantees (up to 1000 VMs) by spreading VMs across fault domains in a region or within an Availability Zone. This enables you to scale out your application while maintaining fault domain isolation that is essential to run quorum-based or stateful workloads, including:

### Scale out with standard Azure virtual machines

Virtual Machine Scale Sets in Flexible Orchestration mode manage standard Azure VMs. You have full control over the virtual machine lifecycle, as well as network interfaces and disks using the standard Azure VM APIs and commands. Individual instances are compatible with the standard Azure IaaS VM API commands, Azure management features such as Azure Resource Manager resource tagging RBAC permissions, Azure Backup, or Azure Site Recovery.


### Instance naming
When you create a VM and add it to a Flexible scale set, you have full control over instance names within the Azure Naming convention rules. When VMs are automatically added to the scale set via autoscaling, you provide a prefix and Azure appends a unique number to the end of the name.

### Automatic instance repairs

Enabling automatic instance repairs for Azure Virtual Machine Scale Sets helps achieve high availability for applications by maintaining a set of healthy instances. The Application Health extension or Load balancer health probes may find that an instance is unhealthy. Automatic instance repairs will automatically perform instance repairs by deleting the unhealthy instance and creating a new one to replace it. Automatic instance repair feature relies on health monitoring of individual instances in a scale set. VM instances in a scale set can be configured to emit application health status using either the Application Health extension or Load balancer health probes. If an instance is found to be unhealthy, then the scale set performs repair action by deleting the unhealthy instance and creating a new one to replace it.

*Terminate notification and automatic repairs* If the terminate notification feature is enabled on a scale set, then during automatic repair operation, the deletion of an unhealthy instance follows the terminate notification configuration. A terminate notification is sent through Azure metadata service – scheduled events – and instance deletion is delayed during the configured delay timeout. However, the creation of a new instance to replace the unhealthy one doesn't wait for the delay timeout to complete.

### Workload
- Structure of the workload
- VMSS scaling, availability zones
- Packaging/publishing workload artifacts
  
### Management


##### Azure Bastion

The solution implements [Azure Bastion](/azure/bastion/bastion-overview) that allows you to connect to virtual machines in the frontend or backend subnets using your browser and the Azure portal, or via the native SSH or RDP client already installed on your local computer. The Transport Layer Security (TLS) protocol protects the connection.

For additional security, you could use Azure Bastion to connect to a jumpbox that's inside your workload's network environment in Azure. In this scenario, the jump box resides in the spoke virtual network, together with the rest of the workload resources. Additional Network Security rules can be implemented to ensure the jumpboxes could only be accessed from the Azure Bastion subnet and in turn, which resources can be accessed from the jumpbox. 

Additionally, you can use just-in-time (JIT), a feature of Microsoft Defender for Cloud. The JIT access feature uses network security groups or Azure Firewall to block all inbound traffic to your jump box. If a user tries to connect to the jump box with appropriate RBAC permissions, this feature configures the network security groups or Azure Firewall to allow inbound access to the selected ports for a specified amount of time. After that time expires, the ports deny all inbound traffic. For more information about JIT access, see [Understanding just-in-time (JIT) VM access](/azure/defender-for-cloud/just-in-time-access-overview?tabs=defender-for-container-arch-aks)
 
##### Managed disks


# [DevOps](#tab/devops)

### OS patching
### Packaging/publishing workload artifacts
### Guest OS config

### Infrastructure as Code (IaC)

Choose an idempotent declarative method over an imperative approach, where possible. Instead of writing a sequence of commands that specify configuration options, use declarative syntax that describes the resources and their properties. One option is an [Azure Resource Manager (ARM)](/azure/azure-resource-manager/templates/overview) templates. Another is Terraform.

Make sure as you provision resources as per the governing policies. For example, when selecting the right VM sizes, stay within the cost constraints, availability zone options to match the requirements of your application.

If you need to write a sequence of commands, use [Azure CLI](/cli/azure/what-is-azure-cli). These commands cover a range of Azure services and can be automated through scripting. Azure CLI is supported on Windows and Linux. Another cross-platform option is Azure PowerShell. Your choice will depend on preferred skillset.

Store and version scripts and template files in your source control system.


# [Identity and Access Management](#tab/iam)

### Managed identities
### Authorization for solution components
### Role based access control (RBAC)


# [Monitoring](#tab/monitoring)

:::image type="content" source="./media/iaas-baseline-monitoring.png" alt-text="IaaS network data flow  diagram" lightbox="./media/iaas-baseline-monitoring.png":::
*Download a [Visio file](https://arch-center.azureedge.net/xxx.vsdx) of this architecture.*

### VM insights (will we using other insights?)
### Workload metrics and instrumentation
### Health probes
### Platform metrics
### Logs
### Log analytic workspace

# [Networking](#tab/networking)

This architecture uses a hub-spoke network topology. The hub and spoke(s) are deployed in separate virtual networks connected through [peering](/azure/virtual-network/virtual-network-peering-overview). Some advantages of this topology are:

- Segregated management. Enables a way to apply governance and adhere to the principle of least privilege. It also supports the concept of an [Azure landing zone](/azure/cloud-adoption-framework/ready/landing-zone/) with separation of duties.

- Minimizes direct exposure of Azure resources to the public internet.

- Organizations often operate with regional hub-spoke topologies. Hub-spoke network topologies can be expanded in the future and provide workload isolation.

- All web applications should require a web application firewall (WAF) service to help govern HTTP traffic flow.

- A natural choice for workloads that span multiple subscriptions.

- It makes the architecture extensible. To accommodate new features or workloads, new spokes can be added instead of redesigning the network topology.

- Certain resources, such as a firewall and DNS can be shared across networks.

- Aligns with the [Azure enterprise-scale landing zones](/azure/cloud-adoption-framework/ready/enterprise-scale/implementation).

:::image type="content" source="./media/iaas-baseline-network-topology.png" alt-text="IaaS baseline architectural diagram" lightbox="./media/iaas-baseline-network-topology.png":::
*Download a [Visio file](https://arch-center.azureedge.net/xxx.vsdx) of this architecture.*

- Update image to show overall network topology with components for each subsection below
- Q: Do we need to add Azure DDoS Protection?

For additional information, see [Hub-spoke network topology in Azure](../../hybrid-networking/hub-spoke.yml).

### Hub

The hub virtual network is the central point of connectivity and observability. A hub always contains an Azure Firewall with global firewall policies defined by your central IT teams to enforce organization wide firewall policy, Azure Bastion, a gateway subnet for VPN connectivity, and Azure Monitor for network observability.

Within the network, three subnets are deployed.

##### Subnet to host Azure Firewall

[Azure Firewall](/azure/firewall/) is firewall as a service. The firewall instance secures outbound network traffic. Without this layer of security, this traffic might communicate with a malicious third-party service that could exfiltrate sensitive company data. [Azure Firewall Manager](/azure/firewall-manager/overview) enables you to centrally deploy and configure multiple Azure Firewall instances and manage Azure Firewall policies for this *hub virtual network* network architecture type.

##### Subnet to host a gateway

This subnet is a placeholder for a VPN or ExpressRoute gateway. The gateway provides connectivity between the routers in your on-premises network and the virtual network.

##### Subnet to host Azure Bastion

This subnet is a placeholder for [Azure Bastion](/azure/bastion/bastion-overview). You can use Bastion to securely access Azure resources without exposing the resources to the internet. This subnet is used for management and operations only.

##### Subnet to host Private Link endpoints (platform)

//TODO: similar to the private endpoint subnet for the spoke but this one is linking global resources that can be accessed from the hub and all peered spokes

//TODO: need guidance to address potential issue when there are DNS conflicts between private DNS names in the spokes and the hub. For example: when we want a platform Key Vault instance for platform certificates
//      and also workload managed Key Vault resources for application secrets
### Spoke

The spoke virtual network contains the AKS cluster and other related resources. The spoke has four subnets:

##### Subnet to host Azure Application Gateway

Azure [Application Gateway](/azure/application-gateway/overview) is a web traffic load balancer operating at Layer 7. The reference implementation uses the Application Gateway v2 SKU that enables [Web Application Firewall](/azure/application-gateway/waf-overview) (WAF). WAF secures incoming traffic from common web traffic attacks, including bots. The instance has a public frontend IP configuration that receives user requests. By design, Application Gateway requires a dedicated subnet.

##### Subnet to host the frontend VM resources

//TODO: the frontend Flexible VMSS, hosting the Web component of the sample workload, lives here
##### Subnet to host the backend VM resources

//TODO: the backend Flexible VMSS, hosting the Api component of the sample workload, lives here

##### Subnet to host Private Link endpoints

Azure Private Link connections are created for the [Azure Container Registry](/azure/container-registry/) and [Azure Key Vault](/azure/key-vault/general/overview), so these services can be accessed using [private endpoint](/azure/private-link/private-endpoint-overview) within the spoke virtual network. Private endpoints don't require a dedicated subnet and can also be placed in the hub virtual network. In the baseline implementation, they're deployed to a dedicated subnet within the spoke virtual network. This approach reduces traffic passing the peered network connection, keeps the resources that belong to the cluster in the same virtual network, and allows you to apply granular security rules at the subnet level using network security groups.

For more information, see [Private Link deployment options](../../../guide/networking/private-link-hub-spoke-network.yml#decision-tree-for-private-link-deployment).

### Plan the IP addresses

//TODO: DO we need a "Plan the IP addresses" section (exists in other baseline)

### Network flow


Network flow, in this context, can be categorized as:

- **Ingress traffic**. From the client to the workload running in the virtual machines.

- **Egress traffic**. From a workload virtual machine to location outside of Azure.

- **Traffic within workload**. Communication workload resources. This traffic includes communication between the various virtual machines and other Azure resources like Key Vault. Also, if your workload accesses backend services and databases integrated with the VNet as private endpoints, communication with the private endpoints would fall into this category.

//TODO: revise statement about connection with services exposed as private endpoints, is confusing. Also, consider if we need to cover explicitly communicating with other
// backend services not exposed as private endpoints

- **Management traffic**. Traffic that goes between the client and the virtual machines.
 
:::image type="content" source="./media/iaas-baseline-network-traffic.png" alt-text="IaaS network data flow  diagram" lightbox="./media/iaas-baseline-network-traffic.png":::
*Download a [Visio file](https://arch-center.azureedge.net/xxx.vsdx) of this architecture.*

##### Traffic to/from internet

The architecture only accepts TLS encrypted requests from the client. TLS v1.2 is the minimum allowed version with a restricted set of cyphers. Server Name Indication (SNI) strict is enabled. End-to-end TLS is set up through Application Gateway by using two different TLS certificates, as shown in this image.

![TLS termination](./media/iaas-baseline-tls-termination.png)

*Download a [Visio file](https://arch-center.azureedge.net/xxxx.vsdx) of this architecture.*

//TODO: review ficticious names used to reference to the workload components

1. The client sends an HTTPS request to the domain name: app.contoso.com. That name is associated with through a DNS A record to the public IP address of Azure Application Gateway. This traffic is encrypted to make sure that the traffic between the client browser and gateway cannot be inspected or changed.

2. Application Gateway has an integrated web application firewall (WAF) and negotiates the TLS handshake for app.contoso.com, allowing only secure ciphers. Application Gateway is a TLS termination point, as it's required to process WAF inspection rules, and execute routing rules that forward the traffic to the configured backend. The TLS certificate is stored in Azure Key Vault. It's accessed using a user-assigned managed identity integrated with Application Gateway. For information about that feature, see [TLS termination with Key Vault certificates](/azure/application-gateway/key-vault-certs).

3. As traffic moves from Application Gateway to the backend, it's encrypted again with another TLS certificate (wildcard for \*.workload.contoso.com) as it's forwarded to one of the frontend VMs. This re-encryption makes sure traffic that is not secure doesn't flow into the workload. Communication between the frontend and the backend components of the solution is also encrypted using the same wildcard certificate in order to ensure end-to-end TLS traffic all at every hop the way through to the workload.

4. The certificates are stored in Azure Key Vault. For more information, see [Add secret management](#add-secret-management).

//TODO: Add content to explain how the certificates are deployed to the VMs. Tentatively we'll be using the Key Vault VM extension that should allow us to integrate directly

##### Traffic to/from private network and on-premises
##### Traffic routing within workload

### Management traffic

//TODO: how to secure control plane network traffic

##### Traffic control

##### Network Security Groups (NSG)

Use network security group rules to restrict traffic between tiers. In this architecture, the following rules are implemented.

1. Deny all inbound traffic from the virtual network. (Use the VIRTUAL_NETWORK tag in the rule.)
1. Allow only inbound traffic to the frontend subnet from the Application Gateway subnet.
1. Allow only inbound traffic to the backend load balancer subnet from the frontend subnet.
1. Allow only inbound traffic to the backend VMs from the backend load balancer subnet.
1. Allow only outbound traffic from the Application Gateway subnet to the frontend and private endpoints subnets.
1. Allow only outbound traffic from the frontend subnet to the backend load balancer and private endpoint subnets.
1. Allow only outbound traffic from the backend load balancer to the backend subnet.
1. Allow only outbound traffic from the backend subnet to the private endpoint subnet.

//TODO: review these rules and add details from the actual implementation
##### Application Security Groups (ASG)

[Application security groups (ASG)](/azure/virtual-network/application-security-groups) enable you to configure network security as a natural extension of an application's structure, allowing you to group virtual machines and define network security policies based on those groups. You can reuse your security policy without individually referencing IP addresses. They also make rules easier to read when reviewing the rules within a NSG.

Two ASGs are used in this scenario:

- WebFrontend ASG - The network interface of the frontend VMs are assigned to this ASG. The AGS is referenced in NSGs to filter traffic to and from the frontend VMs.
- ApiBackend ASG - The network interface of the backend VMs are assigned to this ASG. The AGS is referenced in NSGs to filter traffic to and from the frontend VMs.

//TODO: review these rules and add details from the actual implementation

### Firewall

### NIC/IPConfig + VM lifecycle
### Accelerated Networking
### Private DNS resolution

### Health probes

Application Gateway and Load Balancer both use health probes to monitor the availability of VM instances.

- Application Gateway always uses an HTTP probe.
- Load Balancer can probe with either HTTP or TCP. Generally, if a VM runs an HTTP server, use an HTTP probe. Otherwise, use TCP.

If a probe can't reach an instance within a timeout period, the gateway or load balancer stops sending traffic to that VM. The probe continues to check, and returns the VM to the back-end pool when the VM becomes available again. HTTP probes send an HTTP GET request to a specified path and listen for an HTTP 200 response. This path can be the root path ("/"), or a health-monitoring endpoint that implements custom logic to check the health of the application. The endpoint must allow anonymous HTTP requests.

For more information about health probes, see these resources:

- [Load Balancer health probes](/azure/load-balancer/load-balancer-custom-probe-overview)
- [Application Gateway health monitoring overview](/azure/application-gateway/application-gateway-probe-overview)

For considerations about designing a health probe endpoint, see [Health Endpoint Monitoring pattern](../patterns/health-endpoint-monitoring.yml).


# [Secret management](#tab/secret-management)

Encrypt sensitive data at rest and use [Azure Key Vault](https://azure.microsoft.com/services/key-vault) to manage the database encryption keys. Key Vault can store encryption keys in hardware security modules (HSMs). For more information, see [Configure Azure Key Vault Integration for SQL Server on Azure VMs](/azure/azure-sql/virtual-machines/windows/azure-key-vault-integration-configure). We also recommend that you store application secrets, such as database connection strings, in Key Vault.

### Certificate

In this architecture we configure the VMs with the Azure Key Vault extension (see [extension for Linux](/azure/virtual-machines/extensions/key-vault-linux) or [extension for Windows](/azure/virtual-machines/extensions/key-vault-windows)). The Key Vault VM extension provides automatic refresh of certificates stored in an Azure key vault. Specifically, the extension monitors a list of observed certificates stored in key vaults, and, upon detecting a change, retrieves, and installs the corresponding certificates. 

The extension supports certificate content types PKCS #12, and PEM. VM/VMSS must have assigned a user assigned managed identity, and the Key Vault Access Policy must be set with secrets get and list permission for VM/VMSS managed identity to retrieve a secret's portion of the certificate.

### Key rotation

# [Storage](#tab/storage)

Generic guidance based on technology choice

---