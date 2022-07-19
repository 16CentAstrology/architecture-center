This series of articles will help professionals who are familiar with Amazon Elastic Kubernetes Service (EKS) to understand Azure Kubernetes Service (AKS) while highlighting key similarities and differences between the two managed Kubernetes solutions. The articles provide best practices and automated reference implementations to improve the security, compliance, and observability of your Azure Kubernetes Service deployments.

These articles describe:

- Identity and Access management.
- Cluster observability and monitoring.
- Network topologies.
- Storage options.
- Cost management and optimizations.
- Agent node management.

## Similarities and differences

Please note that AKS is not the only way to run containers Azure, just like EKS is one of the options for AWS. For more information, see [Comparing Container Apps with other Azure container options](/azure/container-apps/compare-options). The scope of this series of articles is to compare AWS EKS with [Azure Kubernetes Service](/azure/aks/intro-kubernetes) (AKS). It does not contrast other Azure services such as Azure Container Apps, Azure Red Hat Openshift, Azure Container Instance, or Azure App Service with AWS services like Amazon Elastic Container Service or AWS Fargate. For more information on the different Azure services you can use to host your containerized workloads, see [Choose an Azure compute service](/azure/architecture/guide/technology-choices/compute-decision-tree)

## Amazon Elastic Kubernetes Service (EKS) to Azure Kubernetes Service guidance

The following articles provide best practices for the specific design areas:

- [Kubernetes Pod Identity](./iam/pod-identity-content)
- [Cost Management for a Kubernetes Cluster](./cost-management/cost-management-content)
- [Kubernetes Monitoring and Logging](./monitoring/monitoring-content)
- [Secure network access to Kubernetes API](./networking/private-clusters-content)
- [Agent node management](./nodes/node-pools-content.md)
- [Kubernetes Storage options](./storage/storage-content)

## Next Steps

To review and compare Azure and AWS core components review the following articles that compare the platforms' capabilities in these core areas:

- [Azure and AWS accounts and subscriptions](../accounts.md)
- [Compute services on Azure and AWS](../compute.md)
- [Relational database technologies on Azure and AWS](../databases.md)
- [Messaging services on Azure and AWS](../messaging.md)
- [Networking on Azure and AWS](../networking.md)
- [Regions and zones on Azure and AWS](../regions-zones.md)
- [Resource management on Azure and AWS](../resources.md)
- [Multi-cloud security and identity with Azure and AWS](../security-identity.md)
- [Compare storage on Azure and AWS](../storage.md)