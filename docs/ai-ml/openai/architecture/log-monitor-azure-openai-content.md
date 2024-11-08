This solution provides comprehensive logging and monitoring capabilities and enhanced security for organization-level deployments of the Azure OpenAI Service API. The solution enables advanced logging capabilities for tracking API usage and performance. It also employs robust security measures to help protect sensitive data and prevent malicious activity.

## Architecture

:::image type="content" source="_images/azure-openai-monitor-log.svg" alt-text="A diagram that shows an architecture that provides monitoring and logging for Azure OpenAI." lightbox="_images/azure-openai-monitor-log.svg" border="false":::

*Download a [Visio file](https://arch-center.azureedge.net/azure-openai-monitor-log.vsdx) of this architecture.*

### Workflow

1. Client applications access Azure OpenAI endpoints to perform text generation or completion and model training or fine-tuning.
2. Azure Application Gateway provides a single point of entry for clients to the private network that contains the Azure OpenAI models and APIs. For internal applications, the gateway also enables hybrid access from cross-premises clients.
   - An Application Gateway web application firewall (WAF) provides protection against common web vulnerabilities and exploits.
   - Application Gateway health probes monitor the health of the APIs.

    > [!NOTE]
    > The load balancing of stateful operations, such as model fine-tuning and deployments and the inference of fine-tuned models, aren't supported.

3. Azure API Management enables security controls and the auditing and monitoring of Azure OpenAI models.
   - In API Management, Microsoft Entra groups that have subscription-based access permissions grant enhanced security access.
   - Azure Monitor request logging enables auditing for all interactions with the models.
   - Monitoring provides detailed information about Azure OpenAI model usage, key performance indicators (KPIs), and metrics, including prompt information and token statistics for usage traceability.
4. API Management connects to all origin resources via Azure Private Link. This configuration provides enhanced privacy for all traffic by containing it in the private network.
5. This topology also supports [multiple Azure OpenAI instances](/azure/architecture/ai-ml/guide/azure-openai-gateway-multi-backend), so you can scale out API usage to ensure high availability and disaster recovery for the service.
6. For Azure OpenAI model inputs and outputs that exceed the default logging capabilities, API management policies forward requests to Azure Event Hubs and Azure Stream Analytics. These services extract payload information and store it in an Azure data storage service like Azure SQL Database or Azure Data Explorer. This process captures specific data for compliance and auditing purposes without any limits on payload sizing and with minimal effect on performance.

    > [!NOTE]
    > Streaming responses with Azure OpenAI models requires more configuration to capture model completions. This type of configuration isn't covered in this architecture.

### Components

- [Application Gateway](/azure/well-architected/service-guides/azure-application-gateway). An application load balancer that helps ensure that all Azure OpenAI API users get the fastest response and highest throughput for model completions. The application gateway also provides a WAF to help protect against common web vulnerabilities and exploits.
- [API Management](/azure/api-management/api-management-key-concepts). An API management platform for back-end Azure OpenAI endpoint access. It provides monitoring and logging capabilities that aren't available natively in Azure OpenAI. API Management also provides monitoring, logging, and managed access to Azure OpenAI resources.
- [Azure Virtual Network](/azure/virtual-network/virtual-networks-overview). A private network infrastructure in the cloud. It provides network isolation so that all network traffic for models is routed privately to Azure OpenAI. In this architecture, the virtual network provides network isolation for all deployed Azure resources.
- [Azure OpenAI](/azure/well-architected/service-guides/azure-openai). A service that hosts models and provides generative model completion outputs. Azure OpenAI provides its users with access to the GPT large language models (LLMs).
- [Monitor](/azure/azure-monitor/overview). A platform service that provides end-to-end observability for applications. It provides access to application logs via Kusto Query Language. It also enables dashboard reports and monitoring and alerting capabilities. In this architecture, Monitor provides access to API Management logs and metrics.
- [Azure Key Vault](/azure/key-vault/general/overview). A service that enhances security storage for keys and secrets that applications use. Key Vault provides enhanced-security storage for all resource secrets.
- [Azure Storage](/azure/storage/common/storage-introduction). A group of services that provide application storage in the cloud. Storage gives Azure OpenAI access to model training artifacts. It also provides persistence of Azure OpenAI managed artifacts.
- [Event Hubs](/azure/well-architected/service-guides/event-hubs). An event ingestion service that receives and processes events from applications and services. Event Hubs provides a scalable event ingestion service for streaming Azure OpenAI model completions.
- [Stream Analytics](/azure/stream-analytics/stream-analytics-introduction). An event-processing engine that provides real-time data stream processing from Event Hubs. Stream Analytics provides real-time processing of streamed messages.
- [Azure Data Explorer](/azure/data-explorer/data-explorer-overview). A fast and highly scalable data exploration service for log and telemetry data. Azure Data Explorer can store all logged conversations that are sent in streaming mode from the LLM.
- [SQL Database](/azure/well-architected/service-guides/azure-sql-database-well-architected-framework). A managed relational database service that provides a secure, scalable database for storing structured data. SQL Database can store all logged conversations that are sent in streaming mode from the LLM.
- [Microsoft Entra ID](/entra/fundamentals/whatis). A service that enables user authentication and authorization to the application and to platform services that support the application. Microsoft Entra ID provides enhanced-security access, including identity-based access control, to all Azure resources.

### Alternatives

Azure OpenAI provides native logging and monitoring capabilities. You can use this native functionality to track telemetry of the service, but the default Azure OpenAI logging function doesn't track or record inputs and outputs, like prompts, tokens, and models, of the service. These metrics are especially important for compliance and to ensure that the service operates as expected. Also, by tracking interactions with the language models that you deploy to Azure OpenAI, you can analyze how your organization uses the service. Then, you can use that information to identify cost and usage patterns that can inform your decisions on scaling and resource allocation.

The following table compares the metrics that the default Azure OpenAI logging function provides with the metrics that this solution provides.

|Metric |Default Azure OpenAI logging|This solution|
|-|-|-|
|Request count| x| x|
|Data in (size)/data out (size)|  x| x|
|Latency| x |x|
|Token transactions (total)| x| x|
|Caller IP address |x (last octet masked)| x|
|Model utilization || x|
|Token utilization (Prompt/Completion) |x| x|
|Input prompt detail || x |
|Output completion detail|| x |
|Request parameters |x| x|
|Deployment operations |x |x|
|Embedding operations |x| x|
|Embedding text detail | | x|
|Image generation operations |x | x|
|Image generation prompt detail | | x|
|Speech-to-text operations |x | x|
|Assistants API operations |x | x|
|Assistants API prompt detail | | x|

## Scenario details

Large organizations that use generative AI models need to audit and log the use of these models to ensure responsible use and corporate compliance. This solution provides organization-level logging and monitoring capabilities for all interactions with AI models. You can use this information to mitigate harmful use of the models and help ensure that you meet security and compliance standards. The solution integrates with existing APIs for Azure OpenAI with little modification, so you can take advantage of existing code bases. Administrators can also monitor service usage for reporting.

The advantages of using this solution include:

- The comprehensive logging of Azure OpenAI model execution, tracked to the source IP address. Log information includes text that users submit to the model and text that the model receives. These logs help ensure that models are used responsibly and within the approved use cases of the service.
- High availability of the model APIs to ensure that you can meet user requests even if traffic exceeds the limits of a single Azure OpenAI service.
- Role-based access that Microsoft Entra ID manages to ensure that the service applies the principle of least privilege.

**Example query for usage monitoring**

```
ApiManagementGatewayLogs
| where tolower(OperationId) in ('completions_create','chatcompletions_create')
| extend modelkey = substring(parse_json(BackendResponseBody)['model'], 0, indexof(parse_json(BackendResponseBody)['model'], '-', 0, -1, 2))
| extend model = tostring(parse_json(BackendResponseBody)['model'])
| extend prompttokens = parse_json(parse_json(BackendResponseBody)['usage'])['prompt_tokens']
| extend completiontokens = parse_json(parse_json(BackendResponseBody)['usage'])['completion_tokens']
| extend totaltokens = parse_json(parse_json(BackendResponseBody)['usage'])['total_tokens']
| extend ip = CallerIpAddress
| summarize
    sum(todecimal(prompttokens)),
    sum(todecimal(completiontokens)),
    sum(todecimal(totaltokens)),
    avg(todecimal(totaltokens))
    by ip, model
```

Output:

:::image type="content" source="_images/monitor-usage.png" alt-text="A screenshot that shows the output of usage monitoring." lightbox="_images/monitor-usage.png":::

**Example query for prompt usage monitoring**

```
ApiManagementGatewayLogs
| where tolower(OperationId) in ('completions_create','chatcompletions_create')
| extend model = tostring(parse_json(BackendResponseBody)['model'])
| extend prompttokens = parse_json(parse_json(BackendResponseBody)['usage'])['prompt_tokens']
| extend prompttext = substring(parse_json(parse_json(BackendResponseBody)['choices'])[0], 0, 100)
```

Output:

:::image type="content" source="_images/prompt-usage.png" alt-text="A screenshot that shows the output of prompt usage monitoring." lightbox="_images/prompt-usage.png":::

### Potential use cases

Use cases for this solution include:

- The deployment of Azure OpenAI for your organization's internal users to accelerate productivity.
- Quota management for token-based Azure OpenAI APIs.
- [High availability of Azure OpenAI for internal applications](/azure/architecture/ai-ml/guide/azure-openai-gateway-multi-backend).
- Enhanced security for the use of Azure OpenAI within regulated industries.

## Considerations

These considerations implement the pillars of the Azure Well-Architected Framework, which is a set of guiding tenets that can be used to improve the quality of a workload. For more information, see [Microsoft Azure Well-Architected Framework](/azure/well-architected/).

### Reliability

Reliability ensures your application can meet the commitments you make to your customers. For more information, see [Design review checklist for Reliability](/azure/well-architected/reliability/checklist).

This scenario ensures high availability of the language models for your organization's users. Application Gateway provides an effective layer-7 application delivery mechanism to ensure fast and consistent access to applications. You can use API Management to configure, manage, and monitor access to your models. The inherent high availability of platform services like Storage, Key Vault, and Virtual Network ensure high reliability for your application. Finally, multiple instances of Azure OpenAI ensure service resilience if application-level failures occur. These architecture components can help you ensure the reliability of your application at organization scale.

### Security

Security provides assurances against deliberate attacks and the abuse of your valuable data and systems. For more information, see [Design review checklist for Security](/azure/well-architected/security/checklist).

This scenario mitigates the risk of data exfiltration and leakage by implementing best practices for application-level and network-level isolation of your cloud services. All network traffic that contains potentially sensitive data and is input to the model is isolated within a private network. This traffic doesn't traverse public internet routes. You can use Azure ExpressRoute to further isolate network traffic to the corporate intranet and help ensure end-to-end network security.

### Cost Optimization

Cost Optimization is about looking at ways to reduce unnecessary expenses and improve operational efficiencies. For more information, see [Design review checklist for Cost Optimization](/azure/well-architected/cost-optimization/checklist).

To help you explore the cost of running this scenario, we preconfigured all of the services in the Azure pricing calculator. To learn how the pricing might change for your use case, change the appropriate variables to match your expected traffic.

The following three sample cost profiles provide estimates based on the amount of traffic. The estimates assume that a conversation contains approximately 1,000 tokens.

- [Small](https://azure.com/e/98ddb659c31543f0a4e9b95803955aef): For processing 10,000 conversations per month.
- [Medium](https://azure.com/e/53b794faa79740a7aec57dfbafd0eb8c): For processing 100,000 conversations per month.
- [Large](https://azure.com/e/2be9f69894cf45de9683528c23ebaab3): For processing 10 million conversations per month.

## Contributors

*This article is maintained by Microsoft. It was originally written by the following contributors.*

Principal authors:

- [Jake Wang](https://www.linkedin.com/in/jake-wang/) | Cloud Solution Architect – AI / Machine Learning
- [Matthew Felton](https://www.linkedin.com/in/matthewfeltonma/) | Cloud Solution Architect – Infrastructure
- [Shaun Callighan](https://www.linkedin.com/in/shaun-callighan-6007ba5/) | Technical Specialist – App Innovation

Other contributors:

- [Mick Alberts](https://www.linkedin.com/in/mick-alberts-a24a1414/) | Technical Writer

*To see non-public LinkedIn profiles, sign in to LinkedIn.*

## Next steps
- [Generative AI Gateway](../../../ai-ml/guide/azure-openai-gateway-guide.yml)
- [Azure-Samples/openai-python-enterprise-logging (GitHub)](https://github.com/Azure-Samples/openai-python-enterprise-logging)
- [Configure Azure AI Services virtual networks](/azure/ai-services/cognitive-services-virtual-networks)

## Related resources
- [Azure OpenAI: Documentation, quickstarts, API reference](/azure/ai-services/openai/)
- [Protect APIs with Application Gateway and API Management](../../../web-apps/api-management/architectures/protect-apis.yml)
- [AI architecture design](../../../data-guide/big-data/ai-overview.md)
