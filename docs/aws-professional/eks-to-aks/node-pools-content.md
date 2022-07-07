# Manage Kubernetes agent nodes

The following article will compare how to add, update, create, and delete worker nodes in Azure Kubernetes Service (AKS) and Amazon Elastic Kubernetes Service (Amazon EKS).

## Amazon EKS managed node groups

Amazon EKS managed node groups automate the provisioning and lifecycle management of worker nodes (Amazon EC2 instances) for Amazon EKS clusters. AWS allows users to create, update, or terminate nodes for their EKS cluster using the [eksctl](https://docs.aws.amazon.com/eks/latest/userguide/eksctl.html) command-line utility. Node updates and terminations automatically cordon and drain nodes to ensure that applications remain available.

Every managed node is provisioned as part of an [Amazon EC2 Auto Scaling group](https://docs.aws.amazon.com/autoscaling/ec2/userguide/AutoScalingGroup.html) that Amazon EKS operates and controls. The Kubernetes Cluster Autoscaler automatically adjusts the number of worker nodes in a cluster when pods fail or are rescheduled onto other nodes. Each node group can be configured to run across multiple [Availability Zones](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-regions-availability-zones.html#concepts-availability-zones) within a region.

You can add a managed node group to a new or existing EKS cluster using the Amazon EKS console, eksctl, AWS CLI, AWS API, or infrastructure as code tools such as AWS CloudFormation and Terraform. For more information on how to create a managed group with the eksctl command-line utility with or without a YAML template, see [Creating a managed node group](https://docs.aws.amazon.com/eks/latest/userguide/create-managed-node-group.html). Nodes launched as part of a managed node group are automatically tagged for auto-discovery by the [Kubernetes cluster autoscaler](https://docs.aws.amazon.com/eks/latest/userguide/autoscaling.html#cluster-autoscaler). You can apply Kubernetes node labels and taints to worker nodes in a managed node group.

You can create multiple managed node groups within a single EKS cluster for different purposes. For example, you can create one node group with the standard Amazon EKS optimized Amazon Linux 2 AMI for some workloads and another with the GPU variant for workloads that require GPU support.

When managed nodes run an [Amazon EKS optimized Amazon Machine Image (AMI)](https://docs.aws.amazon.com/eks/latest/userguide/eks-optimized-amis.html), Amazon EKS is responsible for building patched versions of the AMI when bugs or issues are reported. However, you are responsible for deploying these patched AMI versions to the managed node groups of your EKS clusters. When managed nodes run a custom AMI, you are responsible for building patched versions of the AMI when bugs or issues are reported and then deploying the AMI. For more information, see [Updating a managed node group](https://docs.aws.amazon.com/eks/latest/userguide/update-managed-node-group.html).

[Amazon Virtual Private Cloud (Amazon VPC)](https://docs.aws.amazon.com/vpc/latest/userguide/what-is-amazon-vpc.html) enables you to launch AWS resources into a virtual network composed of public and private subnets. A [subnet](https://docs.aws.amazon.com/vpc/latest/userguide/configure-subnets.html) is a range of IP addresses in the VPC. A public subnet should be used for hosting resources that must be connected to the internet, while a private subnet should be used for hosting those resources that won't be connected to the public internet. Amazon EKS managed node groups can be provisioned in both public and private subnets.

There are no additional costs to use Amazon EKS managed node groups, you only pay for the AWS resources you provision. These include Amazon EC2 instances, Amazon EBS volumes, Amazon EKS cluster hours, and any other AWS infrastructure.

When creating a managed node group, you can choose to leverage the [On-Demand or Spot capacity type](https://docs.aws.amazon.com/eks/latest/userguide/managed-node-groups.html#managed-node-group-capacity-types) to reduce the cost of agent nodes. Amazon EKS deploys a managed node group with an Amazon EC2 Auto Scaling Group that either contains only On-Demand or only Amazon EC2 Spot Instances:

- With On-Demand Instances, you pay for compute capacity by the second, with no long-term commitments.
- Amazon EC2 Spot Instances are spare Amazon EC2 capacity that offers discounts compared to on-Demand prices.
- Amazon EC2 Spot Instances can be interrupted with a two-minute interruption notice when EC2 needs the capacity back.
- Amazon provides Spot Fleet, a method to automate groups of on-demand and spot instances, and Spot Instance Advisor to help predict which region or AZ might provide minimal disruption.
- AWS spot instances' price vary, you pay the price that is in effect for the time period the instance is up, the price is set by Amazon EC2 depending on long-term trends in supply and demand for Spot Instance capacity.

Amazon EKS supports self-managed nodes, which can also be used with Amazon EKS-optimized Linux AMI, but in this case, in contrast to the Managed Nodes Groups, you are responsible for patching and upgrading the AMI and the nodes. It is a best practice to use eksctl, CloudFormation, or infrastructure as code tools to provision self-managed nodes to make it easier for you. You can provision an Auto Scaling group of Windows nodes that register with your Amazon EKS cluster. After the nodes join the cluster, you can deploy Kubernetes applications to them. For more information, see [launching self-managed Windows nodes](https://docs.aws.amazon.com/eks/latest/userguide/launch-windows-workers.html).

Amazon EKS allows running Kubernetes pods on AWS Fargate. Although this topic is not directly related to managing node groups, this feature allows for offloading the execution of part of the pods to AWS Fargate. [AWS Fargate](https://aws.amazon.com/fargate/) is a technology that provides on-demand, right-sized compute capacity for containers. For more information on how to use Fargate with Amazon EKS, see [AWS Fargate](https://docs.aws.amazon.com/eks/latest/userguide/fargate.html)

For more information on managed node groups, see:

- [Managed node groups](https://docs.aws.amazon.com/eks/latest/userguide/managed-node-groups.html)
- [Creating a managed node group](https://docs.aws.amazon.com/eks/latest/userguide/create-managed-node-group.html)
- [Updating a managed node group](https://docs.aws.amazon.com/eks/latest/userguide/update-managed-node-group.html)
- [Deleting a managed node group](https://docs.aws.amazon.com/eks/latest/userguide/delete-managed-node-group.html)
- [Node taints on managed node groups](https://docs.aws.amazon.com/eks/latest/userguide/node-taints-managed-node-groups.html)

## Azure Kubernetes Service cluster architecture

A Kubernetes cluster on Azure is divided into two components as shown by the following picture:

![Azure Kubernetes Service cluster architecture](./media/control-plane-and-nodes.png)

- **Kubernetes Control plane**: provides the core [Kubernetes services](https://kubernetes.io/docs/concepts/overview/components/) and orchestration of application workloads.
- **Nodes**: run your application workloads.

### Control plane

A control plane is automatically created and configured whenever you create an AKS cluster. This control plane is provided at no cost as a managed Azure resource abstracted from the user. You only pay for the nodes attached to the AKS cluster. The control plane and its resources reside only on the region where you created the cluster. For more information, see [Control Plane](/azure/aks/concepts-clusters-workloads#control-plane).

### Nodes and node pools

To run your applications and supporting services, you need a Kubernetes node. An AKS cluster has at least one node, an Azure virtual machine (VM) that runs the Kubernetes node components and container runtime. For more information, see [Nodes and node pools](/azure/aks/concepts-clusters-workloads#nodes-and-node-pools).

![Nodes and node pools](./media/aks-node-resource-interactions.png)

## Azure Kubernetes Service Node Pools

In Azure Kubernetes Service (AKS), nodes of the same configuration are grouped into *node pools*. These node pools contain the underlying virtual machines that run your workloads on AKS. The initial number of worker nodes and their [VM size](/azure/virtual-machines/sizes) are defined when you create an AKS cluster or add a new node pool to an existing AKS cluster. If no VM size is specified when you add a node pool, the default size is *Standard_D2s_v3* for Windows node pools and *Standard_DS2_v2* for Linux node pools. Every AKS cluster must contain at least one [system node pool](/azure/aks/use-system-pools) with at least one node. You can create specialized user node pools to support applications with different compute or storage demands. System node pools serve the primary purpose of hosting critical system pods such as CoreDNS. User node pools serve the primary purpose of hosting your workload pods. However, application pods can be scheduled on system mode node pools if you wish to only have one node pool in your AKS cluster, for example, in your development environment. Vice versa, you can create multiple user node pools to segregate different workloads on different nodes and avoid the [Noisy Neighbor problem](/azure/architecture/antipatterns/noisy-neighbor/noisy-neighbor) or if you want to run compute-intensive applications on GPU-enabled or memory-optimized virtual machines. Every agent node of a system or user node poll is a virtual machine provisioned as part of an [Azure Virtual Machine Scale Set (VMSS)](/azure/virtual-machine-scale-sets/overview) managed by the AKS cluster.

### Create and manage multiple node pools

You can add a node pool to a new or existing AKS cluster using the Azure portal, [Azure CLI](/cli/azure/), [AKS REST API](/rest/api/aks/), or infrastructure as code tools such as [Bicep](/azure/azure-resource-manager/bicep/overview?tabs=bicep), [ARM templates](/azure/azure-resource-manager/templates/overview), or [Terraform](https://www.terraform.io/). For more information on how to add node pools to an existing AKS cluster, see [Create and manage multiple node pools for a cluster in Azure Kubernetes Service (AKS)](/azure/aks/use-multiple-node-pools). When you create a new node pool, the associated VMSS gets created in the [node resource group](/azure/aks/faq#why-are-two-resource-groups-created-with-aks), an [Azure resource group](/azure/azure-resource-manager/management/overview) that contains all of the infrastructure resources used by an AKS cluster. These resources include the Kubernetes nodes, virtual networking resources, managed identities, and storage. By default, the node resource group has a name like *MC_myResourceGroup_myAKSCluster_region*. AKS automatically deletes the [node resource group](/azure/aks/faq#why-are-two-resource-groups-created-with-aks) whenever the cluster is deleted, so it should only be used for resources that share the cluster's lifecycle.

### Add a node pool

The following code snippet shows how to add a node pool named *mynodepool* with 3 nodes to an existing AKS cluster called *myAKSCluster* in the *myResourceGroup* resource group in the using the [az aks nodepool add](/cli/azure/aks/nodepool#az_aks_nodepool_add) command:

  ```bash
  az aks nodepool add \
        --resource-group myResourceGroup \
        --cluster-name myAKSCluster \
        --name mynodepool \
        --node-count 3
  ```

> **Important**: when building an AKS cluster or adding a new node pool to an existing cluster, select a VM series that supports both [Accelerated Networking](/azure/virtual-network/create-vm-accelerated-networking-cli) which provides better latency for intra-node calls and communications with PaaS services.

### Upgrade a node pool

The commands below explain how to upgrade a single specific node pool. You can use the [az aks nodepool upgrade](/cli/azure/aks/nodepool#az_aks_nodepool_upgrade) to upgrade a node pool. To see the available upgrades use [az aks get-upgrades](/cli/azure/aks#az_aks_get_upgrades).

   ```bash
    az aks get-upgrades --resource-group myResourceGroup --name myAKSCluster
  ```

You can use the [az aks nodepool upgrade](/cli/azure/aks/nodepool#az_aks_nodepool_upgrade) command to upgrade a node pool, as shown in the following example:

  ```bash
    az aks nodepool upgrade \
        --resource-group myResourceGroup \
        --cluster-name myAKSCluster \
        --name mynodepool \
        --kubernetes-version KUBERNETES_VERSION \
        --no-wait
  ```

You can list the status of your node pools again using the [az aks node pool list](/cli/azure/aks/nodepool#az_aks_nodepool_list) command.

  ```bash
    az aks nodepool list -g myResourceGroup --cluster-name myAKSCluster
  ```

For more information on how to upgrade the Kubernetes version of the control plane and node pools of an existing cluster, see the following resources:

- [Azure Kubernetes Service (AKS) node image upgrade](/azure/aks/node-image-upgrade)
- [Upgrade a cluster control plane with multiple node pools](/azure/aks/use-multiple-node-pools#upgrade-a-cluster-control-plane-with-multiple-node-pools)

Here is a list of best practices to follow when upgrading a cluster control plane with one or more node pools:

- Make sure to manually or automatically upgrade your AKS cluster to the latest Kubernetes version by setting an auto-upgrade channel on your cluster. For more information, see [Upgrade an Azure Kubernetes Service (AKS) cluster](/azure/aks/upgrade-cluster).
- An AKS cluster upgrade triggers a cordon and drain of your nodes. If you have a low compute quota available, the upgrade may fail. For more information, see [Increase regional vCPU quotas](/azure/azure-portal/supportability/regional-quota-requests).
- Configure the max surge parameter based on your needs using an integer or a percentage value. For production node pools, we recommend a max-surge setting of 33%. For more information, see [Customize node surge upgrade](/azure/aks/upgrade-cluster#customize-node-surge-upgrade).
- When upgrading an AKS cluster that uses Azure CNI, validate there are available private IP address in the subnet for the additional nodes created by the upgrade process based on the max surge setting. For more information, see [Configure Azure CNI networking in Azure Kubernetes Service (AKS)](/azure/aks/configure-azure-cni).
- If you are using Planned Maintenance to patch your virtual machines as well as Auto-Upgrade, your upgrade will start during your specified maintenance window. For more details on Planned Maintenance, see [Use Planned Maintenance to schedule maintenance windows for your Azure Kubernetes Service (AKS) cluster](/azure/aks/planned-maintenance) and [az aks maintenanceconfiguration](/cli/azure/aks/maintenanceconfiguration?view=azure-cli-latest) command.
- If your cluster node pools span across multiple availability zones within a region, the upgrade process can temporarily cause an unbalanced zone configuration. For more information, see [Special considerations for node pools that span multiple Availability Zones](/azure/aks/upgrade-cluster#special-considerations-for-node-pools-that-span-multiple-availability-zones).
- As a best practice, you should upgrade all node pools in an AKS cluster to the same Kubernetes version. The default behavior of the [az aks upgrade](/cli/azure/aks?view=azure-cli-latest#az_aks_upgrade) command is to upgrade all node pools together with the control plane to achieve this alignment.
- Issuing the az aks upgrade command with the --control-plane-only flag upgrades only the cluster control plane. None of the associated node pools in the cluster are changed.
- Upgrading individual node pools requires using the [az aks nodepool upgrade](/cli/azure/aks/nodepool?view=azure-cli-latest#az_aks_nodepool_upgrade) command. This command upgrades only the target node pool with the specified Kubernetes version

### Scale a node pool manually

As your application workload demands change, you may need to scale the number of nodes in a node pool. The number of nodes can be scaled up or down manually. To scale the number of nodes in a node pool, you can use the [az aks node pool scale](/cli/azure/aks/nodepool#az_aks_nodepool_scale) command. The following example scales the number of nodes in mynodepool to 5:

  ```bash
    az aks nodepool scale \
        --resource-group myResourceGroup \
        --cluster-name myAKSCluster \
        --name mynodepool \
        --node-count 5 \
        --no-wait
  ```

You can list the status of your node pools again using the [az aks node pool list](/cli/azure/aks/nodepool#az_aks_nodepool_list) command.

  ```bash
    az aks nodepool list -g myResourceGroup --cluster-name myAKSCluster
  ```

### Scale a specific node pool automatically by enabling the cluster autoscaler

AKS supports scaling node pools in an automatic mode via the  [cluster autoscaler](/azure/aks/cluster-autoscaler). You can enable this feature on each node pool and define a minimum and a maximum number of nodes.

The following [az aks nodepool add](/cli/azure/aks/nodepool?view=azure-cli-latest#az-aks-nodepool-add) command shows how to add a new node pool called *mynodepool* to an existing cluster. The `--enable-cluster-autoscaler` parameter can be used to enable the cluster autoscaler on the new node pool, while the `--min-count` and `--max-count` parameters can be used to indicate, respectively, the minimum and maximum number of nodes.

  ```bash
    az aks nodepool update \
    --resource-group myResourceGroup \
    --cluster-name myAKSCluster \
    --name mynodepool \
    --enable-cluster-autoscaler \
    --min-count 1 \
    --max-count 5
  ```

The following [az aks nodepool update](/cli/azure/aks/nodepool?view=azure-cli-latest#az-aks-nodepool-update) command can be used to update the minimum number of nodes from 1 to 3 for the *mynodepool* node pool.

  ```bash
    az aks nodepool update \
    --resource-group myResourceGroup \
    --cluster-name myAKSCluster \
    --name mynodepool \
    --update-cluster-autoscaler \
    --min-count 1 \
    --max-count 5
  ```

The cluster autoscaler can be disabled with az aks nodepool update and passing the --disable-cluster-autoscaler parameter.

  ```bash
    az aks nodepool update \
    --resource-group myResourceGroup \
    --cluster-name myAKSCluster \
    --name mynodepool \
    --disable-cluster-autoscaler
  ```

Suppose you want to re-enable the cluster autoscaler on an existing cluster. In that case, you can re-enable it using the [az aks nodepool update](/cli/azure/aks/nodepool?view=azure-cli-latest#az-aks-nodepool-update) command, specifying the `--enable-cluster-autoscaler`, `--min-count`, and `--max-count` parameters.

For more information on how to use the cluster autoscaler per node pool, see [Automatically scale a cluster to meet application demands on Azure Kubernetes Service (AKS)](/azure/aks/cluster-autoscaler).

### Taints, Labels, and Tags

When creating a node pool, you can add Kubernetes [taints](https://kubernetes.io/docs/concepts/scheduling-eviction/taint-and-toleration/) and [labels](https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/), or [Azure tags](/azure/azure-resource-manager/management/tag-resources?tabs=json) to that node pool. When you add a taint, label, or tag, all nodes within that node pool also get that taint, label, or tag.

To create a node pool with a taint, you can use the [az aks nodepool add](/cli/azure/aks/nodepool#az_aks_nodepool_add) command with the `--node-taints` parameter. Likewise, to label the nodes in a node pool, you can use the `--labels` parameter and specify a list of labels, as shown in the following code snippet.

  ```bash
    az aks nodepool add \
        --resource-group myResourceGroup \
        --cluster-name myAKSCluster \
        --name mynodepool \
        --node-count 1 \
        --node-taints sku=gpu:NoSchedule \
        --labels dept=IT costcenter=9999 \
        --no-wait
  ```

For more information, see [Specify a taint, label, or tag for a node pool](/azure/aks/use-multiple-node-pools?msclkid=f8996edaa9f611ec988702dcd79cd3f4#specify-a-taint-label-or-tag-for-a-node-pool).

### Network Plugins

When creating a new cluster or adding a new node pool to an existing cluster, you have to specify the resource id of a subnet within the cluster [virtual network (VNet)](/azure/virtual-network/virtual-networks-overview) where to deploy the agent nodes. A workload may require splitting a cluster's nodes into separate node pools for logical isolation. This isolation can be achieved with separate subnets, each dedicated to a separate node pool in the cluster. The virtual machines of a node pool will get a private IP address from the associated subnet.

Azure Kubernetes service supports two networking plugins:

- [Kubenet](https://kubernetes.io/docs/concepts/extend-kubernetes/compute-storage-net/network-plugins/#kubenet): this is a very basic, simple network plugin, on Linux only. With kubenet, nodes get a private IP address from the Azure virtual network subnet. Pods receive an IP address from a logically different address space to the Azure virtual network subnet of the nodes. Network address translation (NAT) is then configured so that the pods can reach resources on the Azure virtual network. The source IP address of the traffic is NAT'd to the node's primary IP address. This approach dramatically reduces the number of IP addresses you need to reserve in your network space for pods.
- [Azure CNI](/azure/aks/configure-azure-cni): with [Azure Container Networking Interface (CNI)](https://github.com/Azure/azure-container-networking/blob/master/docs/cni.md), every pod gets an IP address from the subnet and can be addressed and accessed directly. These IP addresses must be unique across your network space, and must be planned in advance. Each node has a configuration parameter for the maximum number of pods that it supports. The equivalent number of IP addresses per node are then reserved up front for that node. This approach requires more planning, and often leads to IP address exhaustion or the need to rebuild clusters in a larger subnet as your application demands grow.

### Dynamic IP allocation

When using Azure CNI, pods get a private IP address from the same subnet of the hosting node pool. A drawback with the traditional CNI is the exhaustion of pod IP addresses as the AKS cluster grows, resulting in the need to rebuild the entire cluster in a bigger subnet. The [dynamic IP allocation](/azure/aks/configure-azure-cni#dynamic-allocation-of-ips-and-enhanced-subnet-support-preview) capability in Azure CNI solves this problem by allotting pod private IP addresses from a subnet separate from the subnet that hosts the node pool. This feature provides the following advantages:

- **Better IP utilization**: IPs are dynamically allocated to cluster Pods from the Pod subnet. This leads to better utilization of IPs in the cluster compared to the traditional CNI solution, which does static allocation of IPs for every node.
- **Scalable and flexible**: Node and pod subnets can be scaled independently. A single pod subnet can be shared across multiple node pools of a cluster or across multiple AKS clusters deployed in the same VNet. You can also configure a separate pod subnet for a node pool.
- **High performance**: Since pod are assigned VNet private IPs, they have direct connectivity to other cluster pods and resources in the VNet. The solution supports very large clusters without any degradation in performance.
- **Separate VNet policies for pods**: Since pods have a separate subnet, you can configure separate VNet policies for them that are different from node policies. This enables many useful scenarios such as allowing internet connectivity only for pods and not for nodes, fixing the source IP for pod in a node pool using a VNet Network NAT, and using [Network Security Groups (NSGs)](/azure/virtual-network/network-security-groups-overview) to filter traffic between node pools.
- **Kubernetes network policies**: Both the Azure Network Policies and Calico work with dynamic IP allocation.

When creating an AKS cluster or adding a new node pool to an existing cluster that use Azure CNI, you can specify the resource id of two separate subnets, one for the nodes and one for the pods. For more information, see [Dynamic allocation of IPs and enhanced subnet support](/azure/aks/configure-azure-cni#dynamic-allocation-of-ips-and-enhanced-subnet-support-preview).

### Ephemeral OS disk

By default, Azure automatically replicates the operating system disk for an virtual machine to Azure storage to avoid data loss should the VM need to be relocated to another host. However, since containers aren't designed to have local state persisted, this behavior offers limited value while providing some drawbacks, including slower node provisioning and higher read/write latency.

By contrast, ephemeral OS disks are stored only on the host machine, just like a temporary disk, and provide lower read/write latency, along with faster node scaling and cluster upgrades.

Like the temporary disk, an ephemeral OS disk is included in the price of the virtual machine, so you incur no additional storage costs.

> **Important**: when a user does not explicitly request managed disks for the OS, AKS will default to ephemeral OS if possible for a given node pool configuration.

When using ephemeral OS, the OS disk must fit in the VM cache. The sizes for VM cache are available in the Azure documentation in parentheses next to IO throughput ("cache size in GiB").

Using the AKS default VM size Standard_DS2_v2 with the default OS disk size of 100GB as an example, this VM size supports ephemeral OS but only has 86GB of cache size. This configuration would default to managed disks if the user does not specify explicitly. If a user explicitly requested ephemeral OS, they would receive a validation error.

If a user requests the same Standard_DS2_v2 with a 60GB OS disk, this configuration would default to ephemeral OS: the requested size of 60GB is smaller than the maximum cache size of 86GB.

Using Standard_D8s_v3 with 100GB OS disk, this VM size supports ephemeral OS and has 200GB of cache space. If a user does not specify the OS disk type, the node pool would receive ephemeral OS by default.

The following [az aks nodepool add](/cli/azure/aks/nodepool?view=azure-cli-latest#az-aks-nodepool-add) command shows how to add a new node pool to an existing cluster with an ephemeral OS disk. The `--node-osdisk-type` parameter is used to set the OS disk type to Ephemeral, while the `--node-osdisk-size` parameter is used to define the size of the OS disk.

  ```bash
    az aks nodepool add \
        --resource-group myResourceGroup \
        --cluster-name myAKSCluster \
        --name mynodepool \
        --node-osdisk-type Ephemeral \
        --node-osdisk-size 48
  ```

For more information on ephemeral OS disks, see [Ephemeral OS](/azure/aks/cluster-configuration#ephemeral-os).

### Windows node pools

Azure Kubernetes Service supports creating and using Windows Server container node pools. Your AKS cluster needs to use the [Azure CNI](/azure/aks/concepts-network#azure-cni-advanced-networking) network plugin to support Windows node pools. For more detailed information to help plan out the required subnet ranges and network considerations, see [configure Azure CNI networking](/azure/aks/configure-azure-cni).

The following [az aks nodepool add](/cli/azure/aks/nodepool?view=azure-cli-latest#az-aks-nodepool-add) command adds a node pool to an existing cluster that can run Windows Server containers.

  ```bash
    az aks nodepool add \
        --resource-group myResourceGroup \
        --cluster-name myAKSCluster \
        --os-type Windows \
        --name mynodepool \
        --node-count 1
  ```

The above command also uses the default subnet in the virtual network used by the AKS cluster. For more information on how building an AKS cluster with a Windows node pool, see [/azure/aks/windows-container-cli](Create a Windows Server container in AKS).

### Spot node pools

A spot node pool is a node pool backed by a [spot virtual machine scale set](/azure/virtual-machine-scale-sets/use-spot). Using spot virtual machines for nodes with your AKS cluster allows you to take advantage of unutilized capacity in Azure at a significant cost saving. The amount of available unutilized capacity will vary based on many factors, including node size, region, and time of day.

When deploying a spot node pool, Azure will allocate the spot nodes if there's capacity available. But there's no SLA for the spot nodes. A spot scale set that backs the spot node pool is deployed in a single fault domain and offers no high availability guarantees. When Azure needs the capacity back, the Azure infrastructure will evict spot nodes, and you will get a 30-second notice before eviction. Be aware that a spot node pool cannot be the cluster's default node pool. A spot node pool can only be used for a secondary pool.

Spot nodes are great for workloads that can handle interruptions, early terminations, or evictions. For example, workloads such as batch processing jobs, development and testing environments, and large compute workloads may be good candidates to be scheduled on a spot node pool. Review the [spot instance's limitations](/azure/virtual-machines/spot-vms#limitations) for further details.

The following [az aks nodepool add](/cli/azure/aks/nodepool?view=azure-cli-latest#az-aks-nodepool-add) command adds a spot node pool to an existing cluster with autoscaling enabled.

  ```bash
    az aks nodepool add \
        --resource-group myResourceGroup \
        --cluster-name myAKSCluster \
        --name mynodepool \
        --priority Spot \
        --eviction-policy Delete \
        --spot-max-price -1 \
        --enable-cluster-autoscaler \
        --min-count 1 \
        --max-count 3 \
        --no-wait
  ```

For more information on spot node pools, see [Add a spot node pool to an Azure Kubernetes Service (AKS) cluster](/azure/aks/spot-node-pool).

### Virtual nodes

You can use virtual nodes to scale out application workloads quickly in your AKS cluster. With virtual nodes, you have quick provisioning of pods, and only pay per second for their execution time. You don't need to wait for Kubernetes cluster autoscaler to deploy new worker nodes to run the additional pods. Virtual nodes are only supported with Linux pods and nodes.

The virtual nodes add-on for AKS, is based on the open-source project [Virtual Kubelet](https://github.com/virtual-kubelet/virtual-kubelet).

Virtual Nodes functionality depends on [Azure Container Instances](/azure/container-instances/). For more information on virtual nodes, see [Create and configure an Azure Kubernetes Services (AKS) cluster to use virtual nodes](/azure/aks/virtual-nodes?msclkid=dd523912ab5b11ec9eaa4f3e842760f0).

## Considerations

The following limitations apply when you create and manage AKS clusters that support multiple node pools:

- See [Quotas, virtual machine size restrictions, and region availability in Azure Kubernetes Service (AKS)](/azure/aks/quotas-skus-regions).
- You can delete system node pools, provided you have another system node pool to take its place in the AKS cluster.
- System pools must contain at least one node, and user node pools may contain zero or more nodes.
- The AKS cluster must use the Standard SKU load balancer to use multiple node pools, the feature is not supported with Basic SKU load balancers.
- The AKS cluster must use virtual machine scale sets for the nodes.
- You can't change the VM size of a node pool after you create it.
- The name of a node pool may only contain lowercase alphanumeric characters and must begin with a lowercase letter. For Linux node pools the length must be between 1 and 12 - characters, for Windows node pools the length must be between 1 and 6 characters.
- All node pools must reside in the same virtual network.
- When creating multiple node pools at cluster create time, all Kubernetes versions used by node pools must match the version set for the control plane. This can be updated after the cluster has been provisioned by using per node pool operations.
- All subnets assigned to any node pool must belong to the same virtual network.
- System pods must have access to all nodes/pods in the cluster to provide critical functionality such as DNS resolution and tunneling kubectl logs/exec/port-forward proxy.
- If you expand your virtual network after creating the cluster you must update your cluster (perform any managed cluster operation but node pool operations don't count) before adding a subnet outside the original cidr. AKS will error out on the agent pool add now though we originally allowed it. If you don't know how to reconcile your cluster file a support ticket.

## Next Steps

The following references provide links to automation samples and documentation to deploy AKS clusters with a secured API:

- [AKS cluster best practices](//azure/aks/best-practices)
- [GitOps for Azure Kubernetes Service](/azure/architecture/example-scenario/gitops-aks/gitops-blueprint-aks)
- [Create a Private AKS cluster with a Public DNS Zone](https://github.com/Azure/azure-quickstart-templates/tree/master/demos/private-aks-cluster-with-public-dns-zone)
- [Create a private Azure Kubernetes Service cluster using Terraform and Azure DevOps](https://github.com/azure-samples/private-aks-cluster-terraform-devops)
- [Create a public or private Azure Kubernetes Service cluster with Azure NAT Gateway and Azure Application Gateway](https://github.com/Azure-Samples/aks-nat-agic)
- [Use Private Endpoints with a Private AKS Cluster](https://github.com/azure-samples/private-aks-cluster)
- [Create an Azure Kubernetes Service cluster with the Application Gateway Ingress Controller](https://github.com/Azure-Samples/aks-agic)
- [Introduction to Kubernetes](/learn/modules/intro-to-kubernetes/)
- [Introduction to Kubernetes on Azure](/learn/paths/intro-to-kubernetes-on-azure/)
- [Implement Azure Kubernetes Service (AKS)](/learn/modules/implement-azure-kubernetes-service/)
- [Develop and deploy applications on Kubernetes](/learn/paths/develop-deploy-applications-kubernetes/)
- [Optimize compute costs on Azure Kubernetes Service (AKS)](/learn/modules/aks-optimize-compute-costs/)

## Related Resources

- [Azure Kubernetes Service (AKS) solution journey](../../reference-architectures/containers/aks-start-here.md)
- [Azure Kubernetes Services (AKS) day-2 operations guide](../../operator-guides/aks/day-2-operations-guide.md)
- [Choosing a Kubernetes at the edge compute option](../../operator-guides/aks/choose-kubernetes-edge-compute-option.md)