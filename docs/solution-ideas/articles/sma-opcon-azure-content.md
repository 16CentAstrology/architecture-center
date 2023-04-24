[!INCLUDE [header_file](../../../includes/sol-idea-header.md)]

This article outlines a solution for using edge AI when you're disconnected from the internet. The solution uses Azure Stack Hub to move AI models to the edge.

*Apache®, [Apache Hadoop](https://hadoop.apache.org), [Apache Spark](http://spark.apache.org), [Apache HBase](http://hbase.apache.org), and [Apache Storm](https://storm.apache.org) are either registered trademarks or trademarks of the Apache Software Foundation in the United States and/or other countries. No endorsement by the Apache Software Foundation is implied by the use of these marks.*

## Architecture

:::image type="content" source="../media/sma-opcon-azure-architecture.png" alt-text="Architecture diagram that shows an AI-enabled application running at the edge with Azure Stack Hub and hybrid connectivity." lightbox="../media/sma-opcon-azure-architecture.png" border="false":::

*Download a [Visio file](https://arch-center.azureedge.net/sma-opcon-azure-architecture.vsdx) of this architecture.*

### Dataflow

1. An OpCon container provides core services, which are deployed within Azure Kubernetes Service. Persistent volumes (Storage Class Azurefile) are used to store logs and configuration information. These volumes provide data persistence across container restarts. Solution Manager is a web-based user interface that's part of OpCon core services. Users can interact with the entire OpCon environment by using Solution Manager.

1. An Azure SQL database serves as the OpCon database. The core services have secure access to this database through Azure Private Endpoint.

1. OpCon core services use OpCon connector technology to interact with Azure Storage and manage data in Blob Storage. OpCon Managed File Transfer (MFT) also provides support for Azure Storage.

1. The application subnet includes the following components:

   - The virtual machines (VMs) that provide the application infrastructure. These VMs and on-premises legacy systems require connections to OpCon core services to manage their workloads. Applications that provide Rest-API endpoints require no additional software to connect to the core services.  
   - An OpCon MFT Server that provides comprehensive file-transfer functionality. Capabilities include compression, encryption, decryption, decompression, file watching, and enterprise-grade automated file routing.  

1. In a hybrid environment, the Gateway subnet uses a site-to-site VPN tunnel to provide a secure connection between the on-premises environment and the Azure cloud environment.

1. The gateway includes a cross-premises IPsec/IKE VPN tunnel connection between the VPN gateway and an on-premises VPN device. All data that passes between the Azure cloud and the on-premises environment is encrypted in this site-to-site private tunnel as it crosses the internet.

1. A local network gateway in the on-premises environment represents the gateway on the other end of the tunnel. The local network gateway holds configuration information that's needed to build a VPN tunnel to the other end and to route traffic from or to on-premises subnets.

1. All user requests are routed via the gateway connection to the OpCon core services environment. Users access the OpCon Solution Manager framework, a web-based user interface for:

   - OpCon administration.
   - OpCon MFT administration.
   - OpCon workflow development, execution, and monitoring.
   - Self service.
   - Vision, the OpCon task dashboard.
   - OpCon MFT Central Application, a dashboard and query application.

1. OpCon agents and application REST API endpoints are installed on legacy systems in the on-premises environment. OpCon core services use the site-to-site connection on the virtual network gateway to communicate with those agents and endpoints.

### Components



## Scenario details



### Potential use cases



## Next steps



## Related resources


