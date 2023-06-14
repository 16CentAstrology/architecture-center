# OpenCensus on Azure with Application Insights

Description: End-to-end monitoring of a distributed system: a case in point with Azure Functions and OpenCensus





This article describes an example distributed architecture that uses Azure Functions for Python, Azure Event Hubs, and Azure Service Bus. Next, you learn about distributed tracing and how it works by using Python code examples. Also learn how to set up monitoring capabilities by using [opencensus-python](https://github.com/census-instrumentation/opencensus-python) and Application Insights. Learn how to leverage the introduced observability.

In this article, we use an example case of a distributed application that requires events and messaging capabilities. Python being a common language in the ecosystem, we will use it with the Opencensus package recommended by Microsoft.

> [!NOTE]
> OpenCensus and OpenTelemetry are merging, but OpenCensus is still the recommended approach to monitor Azure Functions. OpenTelemetry for Azure is in preview and [features aren't available yet.](/azure/azure-monitor/faq#what-s-the-current-release-state-of-features-within-the-azure-monitor-opentelemetry-distro-)

## Architecture

:::image type="content" source="{source}" alt-text="Diagram that shows the implemented architecture divided into three steps: query, process, and upsert.":::

## Workflow

1) Query  
   A timer-triggered Azure function queries a *Contoso* internal API to get the latest sales data once a day. It then uses the [Azure Event Hub output binding](/azure/azure-functions/functions-bindings-event-hubs-output?tabs=in-process%2Cfunctionsv2%2Cextensionv5&pivots=programming-language-python) to send the unstructured data as events.

1) Process  
   Event Hub triggers an Azure function that processes and formats the unstructured data to a pre-defined structure. It then publishes one message to Azure Service Bus per asset that needs to be imported by [using the Azure Service Bus output binding](/azure/azure-functions/functions-bindings-service-bus-output?tabs=in-process%2Cextensionv5&pivots=programming-language-python).

1) Upsert  
   Service Bus triggers an Azure function that consumes messages from the queue and launches an upsert operation in the common company storage.

It's important to consider potential operation failures of this architecture. Some examples include:

- The internal API is unavailable, which leads to an exception that's raised by the query data Azure function in step one of the architecture.
- In step two, the process data Azure function, encounters data falls outside of the conditions or parameters.
- In step three, the upsert data Azure function, fails. After several retries, the messages from the Service Bus queue go in the [dead letter queue](/azure/service-bus-messaging/service-bus-dead-letter-queues), which is a secondary queue that holds messages that can't be processed or delivered to a receiver after a pre-defined number of retries. Then the messages can follow an established automatic process, or they can be handled manually.

## Components

- [Azure Functions](/azure/azure-functions) is a service that provides managed serverless to run your applications.
- [Azure Event Hubs](/azure/event-hubs/event-hubs-about) is a scalable event ingestion service that can receive and process millions of events per second.
- [Azure Service Bus](/azure/service-bus-messaging/service-bus-messaging-overview) is a fully-managed message broker with message queues and publish-subscribe topics.
- [Azure Table Storage](/azure/storage/tables/table-storage-overview) is a service that stores non-relational structured data (also known as structured NoSQL data) in the cloud, providing a key/attribute store with a schemaless design.
- [Application Insights](/azure/azure-monitor/app/app-insights-overview?tabs=net) is a feature of Azure Monitor that monitors applications from development to test to production. Application Insights analyzes how an application performs, and it reviews application execution data to determine the cause of an incident.

## Scenario details

Distributed systems are comprised of multiple loosely coupled components. It can be difficult to understand how the components communicate and to fully perceive the end-to-end journey of a user request.

Like many companies, Contoso needs to ingest on-premises or third-party data in the cloud while also collecting data about their sales by using services and in-house tools. In this architecture, a department at Contoso built an internal API that exposes the unstructured data, and they ingest the data into common storage that contains structured data from all the departments’ storage. The architecture shows how Contoso extracts, processes, and ingests that metadata in the cloud.

When building a system, especially a distributed system, it's important to make it observable. An observable system:

- Provides a holistic view of the health of the distributed application.
- Measures the operational performance of the system.
- Identifies and diagnoses failures to quickly resolve an issue.

### Potential use cases

## Deploy this scenario

In this example, the system is a chain of microservices. Each microservice can fail independently for various reasons. When that happens, you want to understand what happened so you can troubleshoot. It’s helpful to isolate an end-to-end transaction and follow the journey through the app stack, which consists of passages through different services or microservices. This is called **distributed tracing**.

In the following sections, we show how to set up distributed tracing in Contoso’s architecture that you can deploy using the deploy button below.

![Deploy to Azure button](Aspose.Words.2d867fa4-b8e3-4946-86ba-487ab7e6ab80.002.png) ![Text

Description automatically generated with medium confidence](Aspose.Words.2d867fa4-b8e3-4946-86ba-487ab7e6ab80.003.png)

Note: Since we do not have an internal API, for the sake of this guide we replaced in our architecture the call to the API by a read of a file in an Azure storage.

## Distributed tracing

OpenCensus is a set of open-source libraries to allow collection of distributed tracing, metrics and logging telemetry. It is available in [multiple languages](https://opencensus.io/quickstart/). In this article, we will use the Python implementation of OpenCensus.

### Traces and spans

A transaction is represented by a trace, which is a collection of [spans](https://opencensus.io/tracing/span/#span). For example, if the user of an e-commerce website wants to place an order, clicking on the *Purchase* button leads to several subsequent operations, such as:

- A POST request is submitted to the API which then redirects the user to a “waiting page”.
- Writing logs with contextual information.
- An external call to a SaaS to request a billing page.

Each of these operations can be part of a span, which makes the trace a complete description of what happens when the Purchase button is clicked. 

Similarly, in Contoso’s use-case, when the *Query Data* Azure Function is triggered to start the daily ingestion of the sale data, a trace is created containing multiple spans:

- A span to confirm the trigger details.
- A span to query the internal API.
- A span to create and send an event to the Event Hub.

A span can have children spans. For example, the *Query Data* Azure Function invocation can be described by the following trace:

![Image that depicts a complete trace composed of spans and their child spans.](Aspose.Words.2d867fa4-b8e3-4946-86ba-487ab7e6ab80.004.png)

The *sendMessages* span is split in two children spans: *splitToMessages* and *writeToEventHub* spans that are two sub-operations required to send the created messages.

Notice that all spans are children of a *root* span.

The diagram above illustrates that the use of spans enables a description of all parts involved in the Query step  that happens within the *Query Data* Azure Function. However, each Azure Function creates its own trace. That means that an end-to-end pass-through Contoso’s ingestion system is the union of three traces created by the three Azure Functions. That means it should be possible to correlate the three traces along with their telemetry to build the end-to-end journey, hence describing all parts of the use case. This is where the tracer comes to play.

### Tracer and W3C Trace context

A tracer is an object that holds contextual information. We aim to have that contextual information propagated as data transits through the Azure Functions. To do this, the *opencensus-extension-azure-functions* leverages [the W3C Trace context.](https://www.w3.org/TR/trace-context) 

As its official documentation states, the W3C Trace context is a “specification that defines standard HTTP headers and a value format to propagate context information that enables distributed tracing scenarios.” 

A given component of the system can instantiate a tracer with the context of the previous component making the call by reading the traceparent.The format of a trace looks like this:

Traceparent: [version]-[traceId]-[parentId]-[traceFlags]

For instance, when traceparent = 00-0af7651916cd43dd8448eb211c80319c-b7ad6b7169203331-00

base16(version) = 00

base16(traceId) = 4bf92f3577b34da6a3ce929d0e0e4736

base16(parentId) = 00f067aa0ba902b7

base16(traceFlags) = 00

traceId and parentId are the most important fields. In what follows, base16(version) and base16(traceFlags) are set to 00. You can find more details about these fields [here](https://www.w3.org/TR/trace-context/#versioning-of-tracestate). 

**traceId:** Globally unique identifier of a trace. 

**parentId:** Globally unique identifier of a span. This span is part of the trace identified by traceId**.**

That means that thanks to the W3C Trace context format,  

## Tracer in OpenCensus for Azure Functions

We need to use an extension specific to Azure Functions instead of the opencensus package like in other cases (for example [Python Webapps)](https://learn.microsoft.com/en-us/azure/azure-monitor/app/opencensus-python#instrument-with-opencensus-python-sdk-with-azure-monitor-exporters).

Azure Functions offer a variety of input and output bindings, and each has its way of embedding the traceparent. For Contoso, two Azure Functions are triggered by consuming events and messages.

That means that two things need to be done:

1) The context (characterized by the identifier of the trace and identifier of the current span) needs to be embedded in a *traceparent* in W3C Trace Context format. This embedding is dependant on the nature of the output binding. For instance our example uses Azure Event Hubs as a messaging system, and establishing the right trace context in output binding is achieved by embedding the *traceparent* in the sent event(s) as the “Diagnostic-Id” property after being bytes-encoded.

   Besides, two spans can be linked (and do not have to be parent/child). For distributed tracing we want to ensure that the current span points to the next one. This is done by creating a [link](https://opencensus.io/tracing/span/link/).

**Azure Function worker** takes care of both these considerations for you**.**

1) An Azure function in the middle of the end-to-end flow needs to extract the contextual information from the passed on *traceparent* described above. This is where the extension for Azure Functions comes in handy: instead of doing this inside the code of each Azure Function  the OpenCensus extension for Azure Functions implements a pre-invocation hook on the Function App level.

That pre-invocation hook:

1) creates a *span\_context* object holding the information of the previous span triggering the Azure Function, as shown in the diagram further down.
1) Instantiates a tracer that contains this *span\_context* and creates a new trace for the triggered Azure Function.
1) Injects that tracer in the [Azure Function execution context](/azure/azure-functions/functions-reference?tabs=blob#function-app).

To ensure the traces appear in Application Insights, you must call the *configure()* method that instantiates and configures an [Azure Exporter](https://github.com/census-instrumentation/opencensus-python/tree/master/contrib/opencensus-ext-azure#opencensus-azure-monitor-exporters) whose role is to export all telemetry.

Since the extension is an app-level one, all the above is applied to all Azure Functions of a given Function App.

## Understand and structure the code

The code example inside our Azure Function is structured with spans. In Python, you can create an opencensus span using the *with* statement to access the *span\_context* part of the tracer that is injected in the Azure Function execution context, which allows to get the details of the current span and its parents:

```python
    with context.tracer.span("nameSpan"):
        # DO SOMETHING WITHIN THAT SPAN
```

Let us have a look at the code of the Query Data *Azure Function:*

```python
import datetime
import logging

import azure.functions as func
from opencensus.extension.azure.functions import OpenCensusExtension
from opencensus.trace import config_integration

OpenCensusExtension.configure()
config_integration.trace_integrations(['requests'])
config_integration.trace_integrations(['logging'])

def main(timer: func.TimerRequest, outputEventHubMessage: func.Out[str], context: func.Context) -> None:

    utc_timestamp = datetime.datetime.utcnow().replace(
        tzinfo=datetime.timezone.utc).isoformat()

    if timer.past_due:
        logging.info('The timer is past due!')

    logging.info(f"Query Data Azure Function triggerred. Current tracecontext is:      {context.trace_context.Traceparent}")
    with context.tracer.span("queryExternalCatalog"):
        logging.info('querying the external catalog')
        content = {"key_content_1": "thisisavalue1"}
        content = json.dumps(content)

    with context.tracer.span("sendMessage"):
        logging.info('reading the external catalog')

        with context.tracer.span("splitToMessages"):
            # Do sthg
            logging.info('splitting to messages')

        with context.tracer.span("setMessages"): 
            logging.info('sending messages')
            outputEventHubMessage.set(content)

    logging.info('Python timer trigger function ran at %s', utc_timestamp)
```

In the code abode, there are three main points:

1) An Opencensus.configure() call. This needs to be done in only one Azure function per Function App, which configures the Azure Exporter, to export Python telemetry such as logs, metrics and traces to Azure Monitor.

```python
config_integration.trace_integrations(['requests'])
config_integration.trace_integrations(['logging'])
```

Use the OpenCensus “requests” and “logging” [integrations](https://learn.microsoft.com/en-us/azure/azure-monitor/app/opencensus-python-dependency#dependencies-with-requests-integration) to configure telemetry collection from the request and logging modules for HTTP calls.

1) In the code, there are 5 spans:

- a root span that is part of the tracer injected the context before the execution
- *queryExternalCatalog*
- *sendMessage,* and its two children *splitToMessages* and *setMessages*

Then, every time a span is created, the *span\_context* of the tracer is updated, as you can see in the following diagram:

![](Aspose.Words.2d867fa4-b8e3-4946-86ba-487ab7e6ab80.005.png)

Description of the workflow:

- **An Azure Function is triggered**. 
  A traceparent is injected in the tracer context object with a pre-invocation hook, which is called by the Python worker before the function runs. 
- **Function execution**. 
  Next, the OpencensusExtension.configure() method is called, initializing an Azure Exporter, enabling trace writing to Application Insights.
 
**Tracer relationship description.**

- The tracer object of the Azure function context contains a *span\_context* field, that describes the root span. 
- Every time a span is created in code, a new globally unique identifier is created, updating the span\_context property in the tracer object of the execution context. 
- The *span\_context field* itself contains the *trace\_id* and *id* fields. 
- *trace\_id* never gets updated, but *id* is updated to the generated unique identifier.  
- In the diagram, the root span has two children spans: queryExternaApi span and sendMessage span. 
  - The queryExternalApi span has a new span id that is different from the root span id, and the same applies to sendMessageSpan.
  - sendmessage span itself has two children spans called splitToMessages and setMessages. These two children span ids are updated as previously in the span\_context field of tracer object of the context. 
  - To capture the parent/child relationship between a span and its parent, the spans\_list field is used to describe the lineage of spans in a list: within the splitToMessages span, the *spans\_list* field is a list that contains sendMessage span (the parent span) and splitToMessages span (the current one). This parent/child relationship is how we can create the chain of isolated operations, within the execution of one Azure Function.*   

Now that we have the chain of operations inside one Azure Function, we want to chain it to the subsequent operations done by the next Azure Functions.

![Diagram Description automatically generated](Aspose.Words.2d867fa4-b8e3-4946-86ba-487ab7e6ab80.006.png)

In the above image:

- setMessages span is the last span of the Query Data Azure Function and the code within the span sends an event hub message to the Event Hub, that will trigger the subsequent Azure Function. The span\_context field of the context tracer object contains the information related to this span. However, that information is tied to the Query Data Azure function’s context. 
- The Azure Functions worker adds a bytes-encoded *Diagnostic-Id* in the properties of the sent event and creates a [link](https://opencensus.io/tracing/span/link/#:~:text=A%20link%20describes%20a%20cross-relationship%20between%20spans%20in,span%20to%20another%20can%20help%20correlate%20related%20spans.) to the root span of the subsequent Azure Function
- The pre-invocation hook of the subsequent Process Data Azure Function reads the Diagnostic Id, and sets the context adequately, which enables to chain multiple Azure Functions that are executed separately. 

The same passing of context happens when the *Process Data*  Function sends a message to the Service Bus Queue. 

**Leverage the distributed tracing**

When the above monitoring configurations are in place, we can delve into the capabilities offered by Application Insights to query and visualize the end-to-end transactions.

There are several types of telemetry available in Application Insights. Our example code will mainly generate:

1. Request telemetry: emitted when a request like an HTTP call or Azure Function trigger happens.
1. Dependency telemetry: emitted when making a call to another service, either internal or external to Azure
1. Trace telemetry: from logs generated from the Azure Functions Runtime and the Azure Functions.

For example, the entry to Contoso’s system is a timer-trigger for the *Query Data* Azure Function and emits a *request* telemetry. The logging inside the Azure Function emits *trace* telemetry. When the Azure Function writes an event to the Event Hub, a *dependency* telemetry is emitted.

All this telemetry can be visualized in miscellaneous manners (see related resources)

## Contributors

*This article is maintained by Microsoft. It was originally written by the following contributors.*

Principal author:

[Raouf Aliouat](https://fr.linkedin.com/in/raouf-aliouat) | Software Engineer 2

Other contributors:

[Julien Corioland](https://www.linkedin.com/in/juliencorioland) | Principal Software Engineer

[Benjamin Guinebertière](https://fr.linkedin.com/in/benjguin=3&fclid=247c03a9-237c-693e-12ad-1173228268bb&psq=Benjamin+Guineberti%c3%a8re&u=a1aHR0cHM6Ly9mci5saW5rZWRpbi5jb20vaW4vYmVuamd1aW4&ntb=1) | Principal Software Engineering Manager

[Adina Stoll](https://www.linkedin.com/in/adina-stoll) | Software Engineer 2

## Next steps

- [Azure Monitor](/azure/azure-monitor/overview)
- [OpenCensus Azure Monitor Exporters](https://github.com/census-instrumentation/opencensus-python/tree/master/contrib/opencensus-ext-azure)
- [Code with engineering playbook: observability in microservices](https://microsoft.github.io/code-with-engineering-playbook/observability/microservices/)
- An example of a system that leverages the presented approach: [Synchronization framework for metadata ingestion from external catalogs in Microsoft Purview](https://microsoft.sharepoint.com/:w:/t/CSEFTEFY19/ET9i4_ecx3tOnUSXR5P0NfMBSBFZxYq2iC67Lc-tYg2TtQ?e=BdG7pO)
- To learn how to rely on distributed tracing in a bigger use case, see [metadata ingestion from external catalogs in Microsoft Purview.](/azure/architecture/solution-ideas/articles/sync-framework-metadata-ingestion)

## Related resources

Solution code on Azure samples

- [Distributed tracing and telemetry correlation in Azure Application Insights - Azure Monitor | Microsoft Learn](/azure/azure-monitor/app/distributed-tracing-telemetry-correlation)
- [Azure Monitor](/azure/azure-monitor/overview)
- [Observability for Event Stream Processing with Azure Functions, Event Hubs, and Application Insights](https://devblogs.microsoft.com/cse/2021/05/13/observability-for-event-stream-processing-with-azure-functions-event-hubs-and-application-insights/#2-understanding-operation-ids-operation-links-when-working-with-event-hubs)
- [Monitor Azure Functions and Event Hubs](/azure/architecture/serverless/event-hubs-functions/observability)


