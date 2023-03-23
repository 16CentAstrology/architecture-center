This article addresses the scenario of how to expose a PaaS resource over a private endpoint to specific workload in a single region, hub-spoke network architecture provided by Microsoft Azure Virtual WAN.

> [!IMPORTANT]
> This article is part of a series on Azure Private Link and Azure DNS in Virtual WAN and builds on the network topology defined in the scenario guide. Read the [overview page first](./private-link-vwan-dns-guide.yml) to understand the base network architecture and key challenges.

## Scenario

:::image type="complex" source="./images/dns-private-endpoints-vwan-scenario-single-region.svg" lightbox="./images/dns-private-endpoints-vwan-scenario-single-region.svg" alt-text="Diagram showing the single-region architecture."::: 
Diagram showing the single-region challenge.
:::image-end:::
*Figure 1: Single-region scenario for Virtual WAN with Private Link and Azure DNS - the challenge*

This section defines the scenario and redefines the challenge for this scenario (the challenge is the same as the [nonworking example in the overview page](./private-link-vwan-dns-guide.yml#nonworking-scenario)). The initial scenario architecture builds on the [common network topology defined in the overview guide](./private-link-vwan-dns-guide.yml#common-network-topology). The following are the additions and changes:

- There's only one region with one virtual hub.
- There's an Azure Storage account in the region with public network access disabled. This storage account is only intended to be accessed by the single workload in this scenario.
- There's initially a single Azure Virtual Network connected to the virtual hub.
- The virtual network has a workload subnet that contains a virtual machine (VM) client.
- The virtual network contains a private endpoint subnet that contains a private endpoint for the storage account.

### Successful outcome

The Azure Virtual Machine client can connect to the Azure Storage account via the storage account's private endpoint that is in the same virtual network, and all other access to the storage account is blocked.

### Impediment

You need a DNS record in the DNS flow that is able to resolve the fully qualified domain name (FQDN) of the storage account back to the private IP address of the private endpoint. As identified in the [overview](private-link-vwan-dns-guide.yml#key-challenges), the challenge with this is twofold:

1. It isn't possible to link a private DNS zone that maintains the storage accounts necessary DNS records to a virtual hub.
1. You can link a private DNS zone to the workload network, so you might think that would work. Unfortunately, the [baseline architecture](./private-link-vwan-dns-guide.yml#common-network-topology) stipulates that each connected virtual network has DNS servers configured to point to use the Azure Firewall DNS proxy.

Because you can't link a private DNS zone to a virtual hub, and the virtual network is configured to use the Azure Firewall DNS proxy, Azure DNS servers don't have any mechanism to resolve the (FQDN) of the storage account to the private IP address of the private endpoint, resulting in the client receiving an erroneous DNS response.

#### DNS and HTTP flows

Let's visualize the impediment described above in the context of this workload by reviewing the DNS and resulting HTTP request flows.

:::image type="complex" source="./images/dns-private-endpoints-vwan-scenario-single-region-challenge.svg" lightbox="./images/dns-private-endpoints-vwan-scenario-single-region-challenge.svg" alt-text="Diagram showing the single-region challenge."::: 
Diagram showing the single-region challenge.
:::image-end:::
*Figure 2: Single-region scenario for Virtual WAN with Private Link and Azure DNS - the challenge*

**DNS flow**

1. The DNS query for `stgworkload00.blob.core.windows.net` from the client is sent to the configured DNS server which is Azure Firewall in the peered regional hub.
2. Azure Firewall proxies the request to Azure DNS. Because it isn't possible to link a private DNS zone to a virtual hub, Azure DNS doesn't know how to resolve the FQDN to the private endpoint private IP address. It does know how to resolve the FQDN to the public IP address of the storage account, so it returns the storage account's public IP address.

**HTTP flow**

1. With the DNS result in hand, the public IP address of the storage account, the client issues an HTTP request to `stgworkload00.blob.core.windows.net`.
2. The request is sent to the public IP address of the storage account. This fails for a number of reasons:
   - The NSG on the workload subnet may not allow this Internet-bound traffic.
   - The Azure Firewall that is filtering Internet-bound egress traffic likely doesn't have an application rule to support this flow.
   - Even if both the NSG and Azure Firewall did have allowances for this request flow, the Storage account is configured to block all public network access.
   - And ultimately is violates our goal of only allowing access to the storage account via the private endpoint.

## Solution - Virtual hub extension for DNS

A solution to the challenge is to implement a [virtual hub extension](./private-link-vwan-dns-virtual-hub-extension-pattern.yml) for DNS. The single responsibility for the DNS extension is to enable to the use of private DNS zones in an architecture with a Virtual WAN hub.

The DNS extension is implemented as a Virtual Network that is peered to virtual hub. It's possible to link a private DNS zone to this virtual network. The extension virtual network contains an Azure DNS Private Resolver that enables services outside of this virtual network like Azure Firewall to query the private zone. The following is a high-level list of the components of a virtual hub extension for DNS, along with some required configuration changes:

- A Virtual Network that is peered with the virtual hub. The configured DNS server for the virtual network is the Azure Firewall securing the virtual hub.
- A DNS Private Resolver in the new virtual network. The DNS Private resolver has an inbound endpoint added.
- A private DNS zone named privatelink.blob.core.windows.net.
  - The private DNS zone contains an A record that maps from the storage account name to the private IP address of the private endpoint for the storage account.
  - The private DNS zone is linked to the new virtual network.
- The Azure Firewall DNS server is configured to point at the DNS Private Resolver's inbound endpoint.

The following diagram illustrates the architecture, along with both the DNS and HTTP flows.

:::image type="complex" source="./images/dns-private-endpoints-vwan-scenario-single-region-works.svg" lightbox="./images/dns-private-endpoints-vwan-scenario-single-region-works.svg" alt-text="Diagram showing the working solution with a Virtual hub extension for DNS.":::
The diagram shows a virtual hub secured by Azure Firewall connected to two virtual networks in a single region. One virtual network contains a DNS Private Resolver. The other virtual network contains a subnet with a VM client and a subnet with a Private Link endpoint. Both virtual networks have the Azure Firewall configured as their DNS server. A private DNS zone is linked to the virtual network containing the resolver and contains an A record with a value of the private IP address of the storage account private endpoint. The diagram shows a DNS flow and an HTTP flow. The DNS flow shows the following steps: 1. A DNS query for the storage account FQDN is sent to Azure Firewall, 2. Azure Firewall forwards the query to its configured DNS server that is the DNS Private Resolver, 3. The DNS Private Resolver proxies to Azure DNS and 4. Azure DNS is aware of the private DNS zone. The HTTP flow shows the client issuing an HTTP request to the Private Link endpoint and connecting to the storage account successfully.
:::image-end:::
*Figure 3: Working solution for single region scenario for Virtual WAN with Private Link and DNS*

**DNS flow for the diagram**

1. A DNS query is issued to the configured DNS server for the virtual network where the query originated. The DNS servers are set to custom with the IP address of 10.100.0.132 added. That address is the private IP address of the Azure Firewall securing the virtual hub so the request is forwarded to the firewall.

    :::image type="content" source="./images/workload-vnet-configured-dns-server.png" lightbox="./images/workload-vnet-configured-dns-server.png" alt-text="Screenshot of the workload VNet showing that DNS servers are set to Custom and the private IP address of the Azure Firewall securing the hub added.":::
    *Figure 4: DNS servers configuration for workload virtual network*

1. Because DNS proxy is enabled on the Azure Firewall, it's listening for DNS requests on port 53. It forwards the query to the configured custom DNS server of 10.200.1.4, which is the private IP address of the DNS Private Resolver input endpoint.

    :::image type="complex" source="./images/firewall-policy-dns-settings.png" lightbox="./images/firewall-policy-dns-settings.png" alt-text="Screenshot of the Azure Firewall policy where DNS Proxy is enabled and the DNS servers are set":::
    Screenshot of the Azure Firewall policy where DNS Proxy is enabled and the DNS servers are set to Custom. The entry points to the private IP address of the DNS Private Resolver input endpoint.
    :::image-end:::
    *Figure 5: DNS configuration in Azure Firewall policy*

1. The DNS Private Resolver queries Azure DNS and receives information about an Azure Private DNS virtual network link.

    :::image type="content" source="./images/private-dns-zone-linked-to-vnet.png" lightbox="./images/private-dns-zone-linked-to-vnet.png" alt-text="Screenshot of the private DNS zone virtual network links showing a link to the DNS extension virtual network.":::
    *Figure 6: Private DNS zone virtual network links*

1. The private DNS zone resolves the FQDN of stgworkload00.blob.core.windows.net to 10.1.2.4, which is the IP address of the private endpoint.

    :::image type="content" source="./images/private-dns-zone-config.png" lightbox="./images/private-dns-zone-config.png" alt-text="Screenshot of the private DNS zone with the A record with name stgworkload00 and value 10.1.2.4":::
    *Figure 7: Private DNS zone with the A record for storage account private endpoint*

**HTTP flow for the diagram**

1. The client issues request to stgworkload00.blob.core.windows.net.
1. Because DNS resolved stgworkload00.blob.core.windows.net to 10.1.2.4, the request is issued to the private endpoint. It's important to understand that, even though Azure Firewall is securing private traffic (see Figure 8), the request doesn't get routed through Azure Firewall because the private endpoint is in the same virtual network as the client.
1. A private connection to the storage account is established through the Private Link service.

## Considerations

### Adding spoke networks

 When adding spoke networks, you need to follow the [constraints defined in the common network topology](./private-link-vwan-dns-guide.yml#common-network-topology). Configure them as follows to ensure they're associated to the Default route table in its regional hub, and Azure Firewall is securing both internet and private traffic.

- When adding a spoke virtual network connection to the virtual hub, configure default routing by applying the following settings:

  - **Associate Route Table**: **Default**
  - **Propagate to none**: **Yes**

- When setting the security configuration for the connection, apply the following settings to ensure Azure Firewall is securing internet and private traffic:
  - **Internet traffic**: **Secured by Azure Firewall**
  - **Private traffic**: **Secured by Azure Firewall**

    :::image type="content" source="./images/virtual-hub-vnet-connection-security-configuration.png" lightbox="./images/virtual-hub-vnet-connection-security-configuration.png" alt-text="Screenshot of the security configuration for the virtual network connections showing internet and private traffic secured by Azure Firewall.":::
    *Figure 8: Virtual hub virtual network connections security configuration*

### Virtual hub extension for DNS

- Make sure you deploy the components of the DNS extension prior to adding any PaaS service you want to configure private endpoint DNS records for.

#### Virtual network

- The virtual network for the DNS extension should only contain the resources required for DNS resolution and nothing else.
- The virtual network for the DNS extension should follow the same configuration guidelines under [Adding spoke networks](#adding-spoke-networks).

#### DNS Private Resolver

Consider the following guidance regarding the DNS Private Resolver in the virtual hub DNS extension.

- There should be one DNS extension with one DNS Private Resolver per region.
- The DNS Private Resolver only requires an inbound endpoint and no outbound endpoints for this scenario. The private IP for the endpoint is what we configure for the custom dns service in the Azure Firewall policy (see figure 5).

    :::image type="content" source="./images/dns-private-resolver-inbound-endpoints.png" lightbox="./images/dns-private-resolver-inbound-endpoints.png" alt-text="Screenshot of the inbound endpoints for the DNS Private Resolver showing one endpoint.":::
    *Figure 9: Inbound endpoints for the DNS Private Resolver*

- Follow the [virtual network restrictions](/azure/dns/dns-private-resolver-overview#virtual-network-restrictions) for the DNS Private Resolver.
- The Network Security Group in the subnet for the DNS Private Resolver should only allow UDP traffic from its regional hub to port 53. You should block all other inbound and outbound traffic.

#### Private DNS zone

Because the Azure DNS Private Resolver is resolving DNS via Azure DNS, Azure DNS is able to pick up any private DNS zones linked to its inbound subnet's virtual network.

- Link the private DNS zone to the virtual hub extension for DNS virtual network.
- Follow the guidance on [managing private DNS zones]().
- If you expect PaaS service owners to manage their own entries, configure RBAC accordingly. See [article on ...] for more considerations.

### Storage account

- Make sure that you've deployed the components of the DNS extension before you add a storage account or any PaaS resource you plan to access via private endpoints.
- Set **Disable public access and use private access** under **Network connectivity** to ensure the storage account can only be accessed via private endpoints.
- Add a private endpoint to the private endpoint subnet in the workload virtual network.
- Logging should be sent to a workload-specific Log Analytics Workspace.

### Private endpoint security

A requirement of this solution is to limit the exposure of this storage account. Once you remove public internet access to your PaaS resource, you should address private networking security.

When Azure Firewall is securing private traffic, spoke-to-spoke connectivity is denied by default. This prevents workloads in other spoke networks from accessing private endpoints in the workload virtual network. As you saw, traffic within the virtual network isn't affected. To control access within the virtual network, and add more granular protection, consider the following network security group (NSG) recommendations.

- Create application security groups (ASGs) to group resources that have similar inbound or outbound access needs. In this scenario, use an ASG for VMs that need to access storage and one for Storage Accounts that need to be accessed.
- Make sure the subnet containing the workload VM has an NSG.
- Make sure the subnet containing the private endpoints has an NSG.

#### NSG rules for subnet containing workload VM

While the following screenshot shows several inbound and outbound NSG rules, only 2 of the outbound rules are interesting for this scenario.

- Outbound rules:
  - Allow compute ASG to access storage account ASG
  - Allow compute ASG to Firewall DNS on port 53

:::image type="content" source="./images/workload-nsg-rules.png" lightbox="./images/workload-nsg-rules.png" alt-text="Picture showing NSG rules for workload subnet.":::
*Figure 10: NSG rules for workload subnet

#### NSG rules for subnet containing private endpoints

- Inbound rules:
  - Allow compute ASG to access storage account ASG
  - Deny all other traffic
- Outbound rules:
  - Deny all traffic

:::image type="content" source="./images/private-endpoint-nsg-rules.png" lightbox="./images/private-endpoint-nsg-rules.png" alt-text="Picture showing NSG rules for private endpoint subnet.":::
*Figure 11: NSG rules for private endpoint subnet

#### Private endpoint security in action

The following image illustrates private endpoint security. The diagram adds another virtual network with a second workload. That workload is not able to access the private endpoint.

:::image type="complex" source="./images/dns-private-endpoints-vwan-scenario-single-region-doesnt-work.svg" lightbox="./images/dns-private-endpoints-vwan-scenario-single-region-doesnt-work.svg" alt-text="Diagram showing workload in second spoke virtual network not able to access private endpoint.":::
The diagram shows a virtual hub secured by Azure Firewall connected to three virtual networks in a single region. One virtual network contains a DNS Private Resolver. The second virtual network contains a subnet with a VM client and a subnet with a Private Link endpoint. The third virtual network contains another workload. All three virtual networks have the Azure Firewall configured as their DNS server. A private DNS zone is linked to the virtual network containing the resolver and contains an A record with a value of the private IP address of the storage account private endpoint. The diagram shows a DNS flow and an HTTP flow. The DNS flow shows the following steps: 1. A DNS query for the storage account FQDN is sent to Azure Firewall, 2. Azure Firewall forwards the query to its configured DNS server that is the DNS Private Resolver, 3. The DNS Private Resolver proxies to Azure DNS and 4. Azure DNS is aware of the private DNS zone. The HTTP flow shows the client in the second spoke virtual network issuing an HTTP request which flows through Azure Firewall. The diagram illustrates that Azure Firewall is not allowing spoke-to-spoke communication. The diagram further shows that the NSG can further be used to block the request.
:::image-end:::
*Figure 12: Working solution for single region scenario for Virtual WAN with Private Link and DNS*

**DNS flow for the diagram**

The DNS flow is exactly the same as in [the solution flow](#solution---virtual-hub-extension-for-dns).

**HTTP flow for the diagram**

1. The client issues request to stgworkload00.blob.core.windows.net.
1. DNS resolves stgworkload00.blob.core.windows.net to 10.1.2.4. The request flows through Azure Firewall because it configured to secure private traffic. Azure Firewall blocks the request to the private endpoint. If you choose not to secure private traffic by Azure Firewall, the NSG on the private endpoint subnet should be used to block unwanted private traffic.

## Related resources

- [What is a private endpoint?](/azure/private-link/private-endpoint-overview)
- [Azure Private Endpoint DNS configuration](/azure/private-link/private-endpoint-dns)
- [Private Link and DNS integration at scale](/azure/cloud-adoption-framework/ready/azure-best-practices/private-link-and-dns-integration-at-scale)
- [Azure Private Link in a hub-and-spoke network](/azure/architecture/guide/networking/private-link-hub-spoke-network)
- [DNS for on-premises and Azure resources](/azure/cloud-adoption-framework/ready/azure-best-practices/dns-for-on-premises-and-azure-resources)
- [Single-region data landing zone connectivity](/azure/cloud-adoption-framework/scenarios/cloud-scale-analytics/eslz-network-considerations-single-region)
- [Use Azure Private Link to connect networks to Azure Monitor](/azure/azure-monitor/logs/private-link-security)
- [Azure DNS Private Resolver](/azure/architecture/example-scenario/networking/azure-dns-private-resolver)
- [Improved-security access to multitenant web apps from an on-premises network](/azure/architecture/example-scenario/security/access-multitenant-web-app-from-on-premises)
- [Network-hardened web application with private connectivity to PaaS datastores](/azure/architecture/example-scenario/security/hardened-web-app)
- [Tutorial: Create a private endpoint DNS infrastructure with Azure Private Resolver for an on-premises workload](/azure/private-link/tutorial-dns-on-premises-private-resolver)
