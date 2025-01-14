This article provides a basic architecture to help you learn how to run chat applications that use [Azure OpenAI Service language models](/azure/ai-services/openai/concepts/models). The architecture includes a client user interface (UI) that runs in Azure App Service and uses prompt flow to orchestrate the workflow from incoming prompts out to data stores to fetch grounding data for the language model. The executable flow deploys to a managed online endpoint that has managed compute. The architecture is designed to operate out of a single region.

> [!IMPORTANT]
> This architecture isn't meant for production applications. It's intended to be an introductory architecture that you can use for learning and proof of concept (POC) purposes. When you design your production enterprise chat applications, see the [Baseline OpenAI end-to-end chat reference architecture](./baseline-openai-e2e-chat.yml), which adds production design decisions to this basic architecture.

> [!IMPORTANT]
> :::image type="icon" source="../../_images/github.svg"::: The guidance is backed by an [example implementation](https://github.com/Azure-Samples/openai-end-to-end-basic) that includes deployment steps for this basic end-to-end chat implementation. You can use this implementation as a foundation for your POC to experience working with chat applications that use Azure OpenAI.

## Architecture

:::image type="complex" source="./_images/openai-end-to-end-basic.svg" lightbox="./_images/openai-end-to-end-basic.svg" alt-text="Diagram that shows a basic end-to-end chat architecture." border= "false":::
    The diagram shows an Azure app service that connects directly to a managed online endpoint that sits in front of managed compute. Two arrows point from the managed compute to Azure AI Search and to Azure OpenAI respectively. The diagram also shows Application Insights, Azure Monitor, Azure Key Vault, Azure Container Registry, and Azure Storage.
:::image-end:::

*Download a [Visio file](https://arch-center.azureedge.net/openai-end-to-end-basic.vsdx) of this architecture.*

### Workflow

1. A user issues an HTTPS request to the app service default domain on azurewebsites.net. This domain automatically points to the App Service built-in public IP address. The Transport Layer Security connection is established from the client directly to App Service. Azure completely manages the certificate.
1. Easy Auth, a feature of App Service, helps ensure that the user who accesses the site is authenticated by using Microsoft Entra ID.
1. The client application code that's deployed to App Service handles the request and presents the user a chat UI. The chat UI code connects to APIs that are also hosted in that same App Service instance. The API code connects to an Azure Machine Learning managed online endpoint to handle user interactions.
1. The managed online endpoint routes the request to Machine Learning managed compute where the prompt flow orchestration logic is deployed.
1. The prompt flow orchestration code runs. Among other things, the logic extracts the user's query from the request.
1. The orchestration logic connects to Azure AI Search to fetch grounding data for the query. The grounding data is added to the prompt that is sent to Azure OpenAI in the next step.
1. The orchestration logic connects to Azure OpenAI and sends the prompt that includes the relevant grounding data.
1. Information about the original request to App Service and the call to the managed online endpoint are logged in Application Insights. This log uses the same Azure Monitor Logs workspace that Azure OpenAI telemetry flows to.

### Prompt flow

The preceding workflow describes the flow for the chat application, but the following list outlines a typical prompt flow in more detail.

> [!NOTE]
> The numbers in this flow don't correspond to the numbers in the architecture diagram.

1. The user enters a prompt in a custom chat UI.
1. The interface's API code sends that text to prompt flow.
1. Prompt flow extracts the user intent, which is either a question or a directive, from the prompt.
1. Optionally, prompt flow determines which data stores hold data that's relevant to the user prompt.
1. Prompt flow queries the relevant data stores.
1. Prompt flow sends the intent, the relevant grounding data, and any history that the prompt provides to the language model.
1. Prompt flow returns the result so that it can be displayed on the UI.

You can implement the flow orchestrator in any number of languages and deploy it to various Azure services. This architecture uses prompt flow because it provides a [streamlined experience](/azure/machine-learning/prompt-flow/overview-what-is-prompt-flow) to build, test, and deploy flows that orchestrate between prompts, back-end data stores, and language models.

### Components

Many of the components of this architecture are the same as the resources in the [basic App Service web application architecture](../../web-apps/app-service/architectures/basic-web-app.yml) because the chat UI is based on that architecture. This section highlights the components that you can use to build and orchestrate chat flows, data services, and the services that expose the language models.

- [Azure AI Foundry](/azure/ai-studio/what-is-ai-studio) is a platform that you can use to build, test, and deploy AI solutions. This architecture uses AI Foundry to build, test, and deploy the prompt flow orchestration logic for the chat application.

  - [AI Foundry hub](/azure/ai-studio/concepts/ai-resources) is the top-level resource for AI Foundry. It's the central place where you can govern security, connectivity, and compute resources for use in your AI Foundry projects. You define connections to resources like Azure OpenAI in the AI Foundry hub. The AI Foundry projects inherit these connections.

  - [AI Foundry projects](/azure/ai-studio/how-to/create-projects) are the environments that you use to collaborate while you develop, deploy, and evaluate AI models and solutions.

- [Prompt flow](/azure/machine-learning/prompt-flow/overview-what-is-prompt-flow) is a development tool that you can use to build, evaluate, and deploy flows that link user prompts, actions through Python code, and calls to language learning models. This architecture uses prompt flow as the layer that orchestrates flows between the prompt, different data stores, and the language model. For development, you can host your prompt flows in two types of runtimes.

  - **Automatic runtime** is a serverless compute option that manages the lifecycle and performance characteristics of the compute. It also facilitates flow-driven customization of the environment. This architecture uses the automatic runtime for simplicity.

  - **Compute instance runtime** is an always-on compute option in which the workload team must choose the performance characteristics. This runtime provides more customization and control of the environment.

- [Machine Learning](/azure/well-architected/service-guides/azure-machine-learning) is a managed cloud service that you can use to train, deploy, and manage machine learning models. This architecture uses [Managed online endpoints](/azure/machine-learning/prompt-flow/how-to-deploy-for-real-time-inference), a feature of Machine Learning that deploys and hosts executable flows for AI applications that are powered by language models. Use managed online endpoints to deploy a flow for real-time inferencing. This architecture uses them as a platform as a service endpoint for the chat UI to invoke the prompt flows that the Machine Learning automatic runtime hosts.

- [Azure Storage](/azure/storage/common/storage-introduction) is a storage solution that you can use to persist the prompt flow source files for prompt flow development.

- [Azure Container Registry](/azure/container-registry/container-registry-intro) is a managed registry service that you can use to build, store, and manage container images and artifacts in a private registry for all types of container deployments. This architecture packages flows as container images and stores them in Container Registry.

## Considerations

These considerations implement the pillars of the Azure Well-Architected Framework, which is a set of guiding tenets that can be used to improve the quality of a workload. For more information, see [Microsoft Azure Well-Architected Framework](/azure/architecture/framework).

This basic architecture isn't intended for production deployments. The architecture favors simplicity and cost efficiency over functionality so that you can learn how to build end-to-end chat applications by using Azure OpenAI. The following sections outline some deficiencies of this basic architecture and describe recommendations and considerations.

### Reliability

Reliability ensures your application can meet the commitments you make to your customers. For more information, see [Overview of the Reliability pillar](/azure/architecture/framework/resiliency/overview).

This architecture isn't designed for production deployments, so the following list outlines some of the critical reliability features that this architecture omits:

- The app service plan is configured for the Basic tier, which doesn't have [Azure availability zone](/azure/reliability/availability-zones-overview) support. The app service becomes unavailable if there are any problems with the instance, the rack, or the datacenter that hosts the instance. Follow the [reliability guidance for App Service instances](/azure/architecture/web-apps/app-service/architectures/baseline-zone-redundant#app-services) as you move toward production.

- Autoscaling for the client UI isn't enabled in this basic architecture. To prevent reliability problems caused by a lack of available compute resources, you need to overprovision resources to always run with enough compute to handle maximum concurrent capacity.

- Machine Learning compute doesn't support [availability zones](/azure/reliability/availability-zones-overview). The orchestrator becomes unavailable if there are any problems with the instance, the rack, or the datacenter that hosts the instance. To learn how to deploy the orchestration logic to infrastructure that supports availability zones, see [zonal redundancy for flow deployments](/azure/architecture/ai-ml/architecture/baseline-openai-e2e-chat#zonal-redundancy-for-flow-deployments) in the baseline architecture.

- Azure OpenAI isn't implemented in a highly available configuration. To learn how to implement Azure OpenAI in a reliable manner, see [Azure OpenAI - reliability](/azure/architecture/ai-ml/architecture/baseline-openai-e2e-chat#azure-openai---reliability) in the baseline architecture.

- AI Search is configured for the Basic tier, which doesn't support [Azure availability zones](/azure/reliability/availability-zones-overview). To achieve zonal redundancy, deploy AI Search with the Standard pricing tier or higher in a region that supports availability zones and deploy three or more replicas.

- Autoscaling isn't implemented for the Machine Learning compute. For more information, see the [reliability guidance](/azure/architecture/ai-ml/architecture/baseline-openai-e2e-chat#reliability) in the baseline architecture.

For more information, see [Baseline Azure OpenAI end-to-end chat reference architecture](./baseline-openai-e2e-chat.yml).

### Security

Security provides assurances against deliberate attacks and the abuse of your valuable data and systems. For more information, see [Overview of the Security pillar](/azure/architecture/framework/security/overview).

This section describes some of the key recommendations that this architecture implements. These recommendations include content filtering and abuse monitoring, identity and access management, and role-based access controls. Because this architecture isn't designed for production deployments, this section also discusses network security. Network security is a key security feature that this architecture doesn't implement.

#### Content filtering and abuse monitoring

Azure OpenAI includes a [content filtering system](/azure/ai-services/openai/concepts/content-filter) that uses an ensemble of classification models to detect and prevent specific categories of potentially harmful content in input prompts and output completions. This potentially harmful content includes hate, sexual, self harm, violence, profanity, and jailbreak (content designed to bypass the constraints of a language model) categories. You can configure the strictness of what you want to filter from the content for each category by using the low, medium, or high options. This reference architecture adopts a stringent approach. Adjust the settings according to your requirements.

Azure OpenAI implements content filtering and abuse monitoring features. Abuse monitoring is an asynchronous operation that detects and mitigates instances of recurring content or behaviors that suggest the use of the service in a manner that might violate the [Azure OpenAI code of conduct](/legal/cognitive-services/openai/code-of-conduct). You can request an [exemption of abuse monitoring and human review](/legal/cognitive-services/openai/data-privacy#how-can-customers-get-an-exemption-from-abuse-monitoring-and-human-review) if your data is highly sensitive or if there are internal policies or applicable legal regulations that prevent the processing of data for abuse detection.

#### Identity and access management

The following guidance expands on the [identity and access management guidance](/azure/architecture/web-apps/app-service/architectures/baseline-zone-redundant#identity-and-access-management) in the App Service baseline architecture. This architecture uses system-assigned managed identities and creates separate identities for the following resources:

- AI Foundry hub
- AI Foundry project for flow authoring and management
- Online endpoints in the deployed flow if the flow is deployed to a managed online endpoint

If you choose to use user-assigned managed identities, you should create separate identities for each of the preceding resources.

AI Foundry projects should be isolated from one another. Apply conditions to project role assignments for blob storage to allow multiple projects to write to the same Storage account and keep the projects isolated. These conditions grant access only to specific containers within the storage account. If you use user-assigned managed identities, you need to follow a similar approach to maintain least privilege access.

Currently, the chat UI uses keys to connect to the deployed managed online endpoint. Azure Key Vault stores the keys. When you move your workload to production, you should use Microsoft Entra managed identities to authenticate the chat UI to the managed online endpoint.

#### Role-based access roles

The system automatically creates role assignments for the system-assigned managed identities. Because the system doesn't know what features of the hub and projects you might use, it creates role assignments to support all of the potential features. For example, the system creates the role assignment `Storage File Data Privileged Contributor` to the storage account for AI Foundry. If you aren't using prompt flow, your workload might not require this assignment.

The following table summarizes the permissions that the system automatically grants for system-assigned identities:

| Identity | Privilege | Resource |
| --- | --- | --- |
| AI Foundry hub | Read/write | Key Vault |
| AI Foundry hub | Read/write | Storage |
| AI Foundry hub | Read/write | Container Registry |
| AI Foundry project | Read/write | Key Vault |
| AI Foundry project | Read/write | Storage |
| AI Foundry project | Read/write | Container Registry |
| AI Foundry project | Write | Application Insights |
| Managed online endpoint | Read | Container Registry |
| Managed online endpoint | Read/write | Storage |
| Managed online endpoint | Read | AI Foundry hub (configurations) |
| Managed online endpoint | Write | AI Foundry project (metrics) |

The role assignments that the system creates might meet your security requirements, or you might want to constrain them further. If you want to follow the principle of least privilege, you need to create user-assigned managed identities and create your own constrained role assignments.

#### Network security

To make it easier for you to learn how to build an end-to-end chat solution, this architecture doesn't implement network security. This architecture uses identity as its perimeter and uses public cloud constructs. Services such as AI Search, Key Vault, Azure OpenAI, the deployed managed online endpoint, and App Service are all reachable from the internet. The Key Vault firewall is configured to allow access from all networks. These configurations add surface area to the attack vector of the architecture.

To learn how to include network as an extra perimeter in your architecture, see the [networking](/azure/architecture/ai-ml/architecture/baseline-openai-e2e-chat#networking) section of the baseline architecture.

### Cost Optimization

Cost Optimization is about looking at ways to reduce unnecessary expenses and improve operational efficiencies. For more information, see [Design review checklist for Cost Optimization](/azure/well-architected/cost-optimization/checklist).

This basic architecture doesn't represent the costs for a production-ready solution. The architecture also doesn't have controls in place to guard against cost overruns. The following considerations outline some of the crucial features that affect cost and that this architecture omits:

- This architecture assumes that there are limited calls to Azure OpenAI. For this reason, we recommend that you use pay-as-you-go pricing instead of provisioned throughput. Follow the [Azure OpenAI cost optimization guidance](/azure/architecture/ai-ml/architecture/baseline-openai-e2e-chat#azure-openai) in the baseline architecture as you move toward a production solution.

- The app service plan is configured for the Basic pricing tier on a single instance, which doesn't provide protection from an availability zone outage. The [baseline App Service architecture](/azure/architecture/web-apps/app-service/architectures/baseline-zone-redundant#app-service) recommends that you use Premium plans with three or more worker instances for high availability. This approach affects your costs.

- Scaling isn't configured for the managed online endpoint managed compute. For production deployments, you should configure autoscaling. The [baseline end-to-end chat architecture](/azure/architecture/ai-ml/architecture/baseline-openai-e2e-chat#zonal-redundancy-for-flow-deployments) recommends that you deploy to App Service in a zonal redundant configuration. Both of these architectural changes affect your costs when you move to production.

- AI Search is configured for the Basic pricing tier with no added replicas. This topology can't withstand an Azure availability zone failure. The [baseline end-to-end chat architecture](/azure/architecture/ai-ml/architecture/baseline-openai-e2e-chat#ai-search---reliability) recommends that you deploy the workload with the Standard pricing tier or higher and deploy three or more replicas. This approach can affect your costs as you move toward production.

- There are no cost governance or containment controls in place in this architecture. Make sure that you guard against ungoverned processes or usage that might incur high costs for pay-as-you-go services like Azure OpenAI.

### Operational Excellence

Operational Excellence covers the operations processes that deploy an application and keep it running in production. For more information, see [Design review checklist for Operational Excellence](/azure/well-architected/operational-excellence/checklist).

#### System-assigned managed identities

This architecture uses system-assigned managed identities for AI Foundry hubs, AI Foundry projects, and for the managed online endpoint. The system automatically creates and assigns identities to the resources. The system automatically creates the role assignments required for the system to run. You don't need to manage these assignments.

#### Built-in prompt flow runtimes

To minimize operational burdens, this architecture uses **Automatic Runtime**. Automatic Runtime is a serverless compute option within Machine Learning that simplifies compute management and delegates most of the prompt flow configuration to the running application's `requirements.txt` file and `flow.dag.yaml` configuration. The automatic runtime is low maintenance, ephemeral, and application driven.

#### Monitoring

Diagnostics are configured for all services. All services except App Service are configured to capture all logs. App Service is configured to capture `AppServiceHTTPLogs`, `AppServiceConsoleLogs`, `AppServiceAppLogs`, and `AppServicePlatformLogs`. During the POC phase, it's important to understand which logs and metrics are available for capture. When you move to production, remove log sources that don't add value and only create noise and cost for your workload's log sink.

We also recommend that you [collect data from deployed managed online endpoints](/azure/machine-learning/concept-data-collection) to provide observability to your deployed flows. When you choose to collect this data, the inference data is logged to Azure Blob Storage. Blob Storage logs both the HTTP request and the response payloads. You can also choose to log custom data.

Ensure that you enable the [integration with Application Insights diagnostics](/azure/machine-learning/how-to-monitor-online-endpoints#using-application-insights) for the managed online endpoint. The built-in metrics and logs are sent to Application Insights, and you can use the features of Application Insights to analyze the performance of your inferencing endpoints.

#### Language model operations

Because this architecture is optimized for learning and isn't intended for production use, operational guidance like GenAIOps is out of scope. When you move toward production, follow the [language model operations guidance](/azure/architecture/ai-ml/architecture/baseline-openai-e2e-chat#language-model-operations) in the baseline architecture.

##### Development

Prompt flow provides a browser-based authoring experience in AI Foundry or through a [Visual Studio Code extension](/azure/machine-learning/prompt-flow/community-ecosystem#vs-code-extension). Both options store the flow code as files. When you use AI Foundry, the files are stored in files in a storage account. When you work in VS Code, the files are stored in your local file system.

Because this architecture is meant for learning, it's okay to use the browser-based authoring experience. When you start moving toward production, follow the [development and source control guidance](/azure/architecture/ai-ml/architecture/baseline-openai-e2e-chat#development) in the baseline architecture.

We recommend that you use the serverless compute option when you develop and test your prompt flows in AI Foundry. This option keeps you from having to deploy and manage a compute instance for development and testing. If you need a customized environment, you can deploy a compute instance.

##### Evaluation

You can conduct an evaluation of your Azure OpenAI model deployment by using user experience in AI Foundry. We recommend that you become familiar with how to [evaluate generative AI applications](/azure/ai-studio/concepts/evaluation-approach-gen-ai) to help ensure that the model that you choose meets customer and workload design requirements.

One important evaluation tool that you should familiarize yourself with during your workload's development phase is the [Responsible AI dashboards in Machine Learning](/azure/machine-learning/how-to-responsible-ai-dashboard?view=azureml-api-2). This tool helps you evaluate the fairness, model interpretability, and other key assessments of your deployments and is useful to help establish an early baseline to prevent future regressions.

##### Deployment

This basic architecture implements a single instance for the deployed orchestrator. When you deploy changes, the new deployment takes the place of the existing deployment. When you start moving toward production, read the [deployment flow](/azure/architecture/ai-ml/architecture/baseline-openai-e2e-chat#deployment-flow) and [deployment guidance](/azure/architecture/ai-ml/architecture/baseline-openai-e2e-chat#deployment-guidance) in the baseline architecture. This guidance helps you understand and implement more advanced deployment approaches, such as blue-green deployments.

### Performance Efficiency

Performance Efficiency is the ability of your workload to meet the demands placed on it by users in an efficient manner. For more information, see [Design review checklist for Performance Efficiency](/azure/well-architected/performance-efficiency/checklist).

Because this architecture isn't designed for production deployments, this section outlines some of the critical performance efficiency features that the architecture omits.

One outcome of your POC should be the selection of a product that suits the workload for your app service and your Machine Learning compute. You should design your workload to efficiently meet demand through horizontal scaling. Horizontal scaling allows you to adjust the number of compute instances that are deployed in the app service plan and in instances that are deployed behind the online endpoint. Don't design a system that depends on changing the compute product to align with demand.

- This architecture uses the consumption or pay-as-you-go model for most components. The consumption model is a best-effort model and might be subject to noisy neighbor problems or other stressors on the platform. Determine whether your application requires [provisioned throughput](/azure/ai-services/openai/concepts/provisioned-throughput) as you move toward production. Provisioned throughput helps ensure that processing capacity is reserved for your Azure OpenAI model deployments. Reserved capacity provides predictable performance and throughput for your models.

- The Machine Learning online endpoint doesn't have automatic scaling implemented, so you need to provision a product and instance quantity that can handle peak load. Because of how the service is configured, it doesn't dynamically scale in to efficiently keep supply aligned with demand. Follow the guidance about how to [autoscale an online endpoint](/azure/machine-learning/how-to-autoscale-endpoints) as you move toward production.

### Additional design recommendations

AI/ML workloads, such as this one, should be designed by an architect that understands the design guidance found in the Azure Well-Architected Framework's [AI workloads on Azure](/azure/well-architected/ai/get-started). As you move from ideation and proof of technology into design, be sure to combine both the specifics of the learnings you have from this architecture and the general AI/ML workload guidance found in the Well-Architected Framework.

## Deploy this scenario

A deployment for a reference architecture that implements these recommendations and considerations is available on [GitHub](https://github.com/Azure-Samples/openai-end-to-end-basic/).

## Next step

> [!div class="nextstepaction"]
> [Baseline Azure OpenAI end-to-end chat reference architecture](./baseline-openai-e2e-chat.yml)

## Related resources

- A Well-Architected Framework perspective on [AI workloads on Azure](/azure/well-architected/ai/get-started)
- [Azure OpenAI language models](/azure/ai-services/openai/concepts/models)
- [Prompt flow](/azure/machine-learning/prompt-flow/overview-what-is-prompt-flow)
- [Content filtering](/azure/ai-services/openai/concepts/content-filter)
