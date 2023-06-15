# OpenCensus on Azure with Application Insights

This article describes a distributed architecture that uses Azure Functions for Python, Azure Event Hubs, and Azure Service Bus to monitor an end-to-end system. It introduces distributed tracing and explains how it works by using Python code examples. You can also learn how to set up monitoring capabilities by using [OpenCensus for Python](https://github.com/census-instrumentation/opencensus-python) and Application Insights.

> [!NOTE]
> OpenCensus and OpenTelemetry are merging, but OpenCensus is still the recommended approach to monitor Azure Functions. OpenTelemetry for Azure is in preview and [some features aren't available yet.](/azure/azure-monitor/faq#what-s-the-current-release-state-of-features-within-the-azure-monitor-opentelemetry-distro-)

## Architecture

:::image type="content" source="{source}" alt-text="Diagram that shows the implemented architecture divided into three steps: query, process, and upsert.":::

## Workflow

1) **Query.** A timer-triggered Azure function queries a *Contoso* internal API to get the latest sales data once a day. The function uses the [Azure Event Hubs output binding](/azure/azure-functions/functions-bindings-event-hubs-output?tabs=in-process%2Cfunctionsv2%2Cextensionv5&pivots=programming-language-python) to send the unstructured data as events.

1) **Process.** Event Hubs triggers an Azure function that processes and formats the unstructured data to a pre-defined structure. The function publishes one message to Service Bus per asset that needs to be imported by [using the Service Bus output binding](/azure/azure-functions/functions-bindings-service-bus-output?tabs=in-process%2Cextensionv5&pivots=programming-language-python).

1) **Upsert.** Service Bus triggers an Azure function that consumes messages from the queue and launches an upsert operation in the common company storage.

## Components

- [Application Insights](/azure/azure-monitor/app/app-insights-overview?tabs=net) is a feature of Azure Monitor that monitors applications from development to test to production. Application Insights analyzes how an application performs, and it reviews application execution data to determine the cause of an incident.
- [Event Hubs](/azure/event-hubs/event-hubs-about) is a scalable event ingestion service that can receive and process millions of events per second.
- [Azure Functions](/azure/azure-functions) is a service that provides managed serverless to run your applications.
- [Service Bus](/azure/service-bus-messaging/service-bus-messaging-overview) is a fully-managed message broker with message queues and publish-subscribe topics.
- [Azure Table Storage](/azure/storage/tables/table-storage-overview) is a service that stores non-relational structured data (also known as structured NoSQL data) in the cloud, providing a key/attribute store with a schemaless design.
- [OpenCensus](https://opencensus.io/quickstart/) is a set of open-source libraries where you can collect distributed traces, metrics, and logging telemetry. This article uses the Python implementation of OpenCensus.

## Considerations

It's important to consider potential operation failures of this architecture. Some examples include:

- The internal API is unavailable, which leads to an exception that's raised by the query data Azure function in step one of the architecture.
- In step two, the process data Azure function, encounters data falls outside of the conditions or parameters.
- In step three, the upsert data Azure function, fails. After several retries, the messages from the Service Bus queue go in the [dead letter queue](/azure/service-bus-messaging/service-bus-dead-letter-queues), which is a secondary queue that holds messages that can't be processed or delivered to a receiver after a pre-defined number of retries. Then the messages can follow an established automatic process, or they can be handled manually.

## Scenario details

Distributed systems are comprised of multiple loosely coupled components. It can be difficult to understand how the components communicate and to fully perceive the end-to-end journey of a user request.

Like many companies, Contoso needs to ingest on-premises or third-party data in the cloud while also collecting data about their sales by using services and in-house tools. In this architecture, a department at Contoso built an internal API that exposes the unstructured data, and they ingest the data into common storage that contains structured data from all the departments’ storage. The architecture shows how Contoso extracts, processes, and ingests that metadata in the cloud.

When building a system, especially a distributed system, it's important to make it observable. An observable system:

- Provides a holistic view of the health of the distributed application.
- Measures the operational performance of the system.
- Identifies and diagnoses failures to quickly resolve an issue.

### Potential use cases

## Deploy this scenario

In this example, the system is a chain of microservices. Each microservice can fail independently for various reasons. When that happens, it's important to understand what happened so you can troubleshoot. It’s helpful to isolate an end-to-end transaction and follow the journey through the app stack, which consists of different services or microservices. This is called distributed tracing.

The following sections show you how to set up distributed tracing in Contoso’s architecture. To follow along, select the deploy button.

![Deploy to Azure button](Aspose.Words.2d867fa4-b8e3-4946-86ba-487ab7e6ab80.002.png) ![Text

Description automatically generated with medium confidence](Aspose.Words.2d867fa4-b8e3-4946-86ba-487ab7e6ab80.003.png)

> [!NOTE]
> In the deployment, the call to the API is replaced by a read of a file in an Azure storage.

### Distributed tracing

**add description** make sure "transaction" is introduced

#### Traces and spans

A transaction is represented by a trace, which is a collection of [spans](https://opencensus.io/tracing/span/#span). For example, when you select the purchase button to place an order on an e-commerce website, several subsequent operations take place. For example:

- A POST request submits to the API which then redirects you to a “waiting page”.
- Writing logs with contextual information.
- An external call to a SaaS to request a billing page.

Each of these operations can be part of a span. The trace is a complete description of what happens when you select the purchase button.

Similarly, in Contoso’s use case, when the query data Azure function triggers to start the daily ingestion of the sales data, a trace is created that contains multiple spans:

- A span to confirm the trigger details.
- A span to query the internal API.  
- A span to create and send an event to Event Hubs.

A span can have children spans. For example, the following image shows the query data Azure function as a trace:

![Image that depicts a complete trace composed of spans and their child spans.](Aspose.Words.2d867fa4-b8e3-4946-86ba-487ab7e6ab80.004.png)

The *sendMessages* span is split into two children spans: *splitToMessages* and *writeToEventHubs*. The *sendMessages* span requires the two sub-operations to send messages.

All spans are children of a *root* span.

In the previous image, spans give you an easy way to describe all parts involved in the query step of the query data Azure function. Each Azure function is a trace. So an end-to-end pass through Contoso’s ingestion system is the union of three traces, which are the three Azure functions. When you combine the three traces and their telemetry, you build the end-to-end journey and describe all parts of the use case.

#### Tracer and W3C trace context

A tracer is an object that holds contextual information. It's ideal to have that contextual information propagated as data transits through the Azure functions. To do this, the *opencensus-extension-azure-functions* leverages [the W3C trace context](https://www.w3.org/TR/trace-context).

As its official documentation states, the W3C trace context is a “specification that defines standard HTTP headers and a value format to propagate context information that enables distributed tracing scenarios.”

A given component of the system can instantiate a tracer with the context of the previous component making the call by reading the traceparent. The format of a trace is:

Traceparent: [*version*]-[*traceId*]-[*parentId*]-[*traceFlags*]

For instance, if traceparent = 00-0af7651916cd43dd8448eb211c80319c-b7ad6b7169203331-00

base16(version) = 00

base16(traceId) = 0af7651916cd43dd8448eb211c80319c

base16(parentId) = b7ad6b7169203331

base16(traceFlags) = 00

The traceId and parentId are the most important fields. In what follows, base16(version) and base16(traceFlags) are set to 00. The traceId is a globally unique identifier of a trace. The parentId is a globally unique identifier of a span. That span is part of the trace that the traceId identifies.

For more information, see [Traceparent header](https://www.w3.org/TR/trace-context/#traceparent-header).

### Tracer in OpenCensus for Azure Functions

Use an extension that's specific to Azure Functions. Don't use the OpenCensus package that you might use in other cases (for example, [Python Webapps](/azure/azure-monitor/app/opencensus-python#instrument-with-opencensus-python-sdk-with-azure-monitor-exporters)).

Azure Functions offers many input and output bindings, and each binding has a different way of embedding the traceparent. For Contoso, when events and messages are consumed, two Azure functions are triggered.

That means that two things need to be done:

1) The context (characterized by the identifier of the trace and identifier of the current span) must be embedded in a *traceparent* in the W3C trace context format. This embedding is dependant on the nature of the output binding. For instance, the example architecture uses Event Hubs as a messaging system. The traceparent is encoded into bytes and embedded in the sent event(s) as the “Diagnostic-Id” property, which achieves the right trace context in the output binding.

   Two spans can be linked even if they're not parent and child. For distributed tracing, the current span points to the next one. To establish this relationship, Creating a [link](https://opencensus.io/tracing/span/link/) establishes this relationship.

   The [Azure Functions Worker](https://www.nuget.org/packages/Microsoft.Azure.Functions.Worker/1.15.0-preview1#readme-body-tab) package manages the embedding and linking for you.

1) An Azure function in the middle of the end-to-end flow extracts the contextual information from the passed on *traceparent*. Use the OpenCensus extension for Azure Functions for this step. Instead of adding this process in the code of each Azure function, the OpenCensus extension implements a pre-invocation hook on the function app level.

   The pre-invocation hook:

   - Creates a *span\_context* object that holds the information of the previous span and triggers the Azure function. See a visual example of this step [in the next section](#understand-and-structure-the-code).
   - Creates a tracer that contains the *span\_context* and creates a new trace for the triggered Azure function.
   - Injects the tracer in the [Azure function execution context](/azure/azure-functions/functions-reference?tabs=blob#function-app).

   To ensure the traces appear in Application Insights, you must call the *configure()* method to create and configure an [Azure exporter](https://github.com/census-instrumentation/opencensus-python/tree/master/contrib/opencensus-ext-azure#opencensus-azure-monitor-exporters), which exports telemetry.

   The extension is at app level, so the previous steps in this section apply to all Azure functions of a function app.

### Understand and structure the code

The code example inside our Azure Function is structured with spans. In Python, create an OpenCensus span by using the *with* statement to access the *span\_context* part of the tracer that's injected in the Azure function execution context. This string provides the details of the current span and its parents:

```python
    with context.tracer.span("nameSpan"):
        # DO SOMETHING WITHIN THAT SPAN
```

The following code shows details of the query data Azure function:

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

    logging.info(f"Query Data Azure Function triggered. Current tracecontext is:      {context.trace_context.Traceparent}")
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

The main points of the code are:

1. An Opencensus.configure() call. Perform this call in only one Azure function per function app. This action configures the Azure exporter to export Python telemetry, such as logs, metrics, and traces to Azure Monitor.

1. Use the OpenCensus “requests” and “logging” [integrations](/azure/azure-monitor/app/opencensus-python-dependency#dependencies-with-requests-integration) to configure the telemetry collection from the request and logging modules for HTTP calls.

   ```python
   config_integration.trace_integrations(['requests'])
   config_integration.trace_integrations(['logging'])
   ```

1. There are five spans:

- A root span that's part of the tracer that's injected in the context before the execution.
- *queryExternalCatalog*
- *sendMessage*
- *splitToMessages* (a child of *sendMessage*)
- *setMessages* (a child of *sendMessage*)

### New heading?

The following diagram shows how every time a span is created, the *span\_context* of the tracer is updated.

![](Aspose.Words.2d867fa4-b8e3-4946-86ba-487ab7e6ab80.005.png)

Description of the workflow:

- **An Azure function is triggered**.
  A traceparent is injected in the tracer context object with a pre-invocation hook, which is called by the Python worker before the function runs.
- **Function execution**.
  Next, the OpencensusExtension.configure() method is called, initializing an Azure Exporter, enabling trace writing to Application Insights.

**Tracer relationship description.**

- The tracer object of the Azure function context contains a *span\_context* field that describes the root span.
- Every time you create a span in code, it creates a new globally unique identifier and updates the *span\_context* property in the tracer object of the execution context.
- The *span\_context* field contains the *trace\_id* and *id* fields.
- The *trace\_id* never gets updated, but the *id* updates to the generated unique identifier.  
- In the diagram, the root span has two children spans: queryExternalApi and sendMessage.
  - The queryExternalApi span and sendMessage span have a new span id that's different from the root_span_id.
  - The sendMessage span has two children spans: splitToMessages and setMessages. Their span ids update in the span\_context field of the tracer object of the context.
  - To capture the relationship between a child span and its parent, the spans\_list field provides the lineage of spans in list form. In the splitToMessages span, the *spans\_list* field contains sendMessage (the parent span) and splitToMessages (the current span). This parent/child relationship is how you create the chain of isolated operations within the execution of an Azure function.

Now that the chain of operations are organized in one Azure function, you can chain it to the subsequent operations performed by the next Azure function.

![Diagram Description automatically generated](Aspose.Words.2d867fa4-b8e3-4946-86ba-487ab7e6ab80.006.png)

In the above image:

- The setMessages span is the last span of the query data Azure function. The code within the span sends a message to Event Hubs and triggers the subsequent Azure function. The span\_context field of the context tracer object contains the information related to this span. That information is tied to the query data Azure function’s context.
- Azure Functions Worker adds a bytes-encoded *Diagnostic-Id* in the properties of the sent event and creates a [link](https://opencensus.io/tracing/span/link/#:~:text=A%20link%20describes%20a%20cross-relationship%20between%20spans%20in,span%20to%20another%20can%20help%20correlate%20related%20spans.) to the root span of the subsequent Azure function.
- The pre-invocation hook of the subsequent process data Azure function reads the Diagnostic Id and sets the context, which chains the Azure functions, and they're executed separately.

When the process data Azure function sends a message to the Service Bus queue, context is passed in the same way.

#### Take advantage of distributed tracing

When the monitoring configurations are in place, use the Application Insights features to query and visualize the end-to-end transactions.

There are several types of telemetry available in Application Insights. The example code generates the following telemetry:

- Request telemetry emits when you call an HTTP or trigger an Azure function.
- Dependency telemetry emits when you make a call to an Azure service or an external service.
- Trace telemetry emits from logs generated by Azure Functions runtime and Azure Functions.

For example, the entry to Contoso’s system has a timer-trigger for the query data Azure function that emits a request telemetry. The logging inside the Azure function emits trace telemetry. When the Azure function writes an event to Event Hubs, it emits a dependency telemetry.

## Contributors

*This article is maintained by Microsoft. It was originally written by the following contributors.*

Principal author:

- [Raouf Aliouat](https://fr.linkedin.com/in/raouf-aliouat) | Software Engineer II

Other contributors:

- [Julien Corioland](https://www.linkedin.com/in/juliencorioland) | Principal Software Engineer
- [Benjamin Guinebertière](https://fr.linkedin.com/in/benjguin=3&fclid=247c03a9-237c-693e-12ad-1173228268bb&psq=Benjamin+Guineberti%c3%a8re&u=a1aHR0cHM6Ly9mci5saW5rZWRpbi5jb20vaW4vYmVuamd1aW4&ntb=1) | Principal Software Engineering Manager
- [Adina Stoll](https://www.linkedin.com/in/adina-stoll) | Software Engineer II

*To see non-public LinkedIn profiles, sign in to LinkedIn.*

## Next steps

- [Azure Monitor](/azure/azure-monitor/overview)
- [OpenCensus Azure Monitor exporters](https://github.com/census-instrumentation/opencensus-python/tree/master/contrib/opencensus-ext-azure)
- [Code with engineering playbook: observability in microservices](https://microsoft.github.io/code-with-engineering-playbook/observability/microservices/)
- An example of a system that leverages the presented approach: [Synchronization framework for metadata ingestion from external catalogs in Microsoft Purview](https://microsoft.sharepoint.com/:w:/t/CSEFTEFY19/ET9i4_ecx3tOnUSXR5P0NfMBSBFZxYq2iC67Lc-tYg2TtQ?e=BdG7pO)
- To learn how to rely on distributed tracing in a bigger use case, see [Metadata ingestion from external catalogs in Microsoft Purview](/azure/architecture/solution-ideas/articles/sync-framework-metadata-ingestion)
- [Distributed tracing and telemetry correlation in Azure Application Insights](/azure/azure-monitor/app/distributed-tracing-telemetry-correlation)
- [Azure Monitor](/azure/azure-monitor/overview)
- [Observability for Event Stream Processing with Azure Functions, Event Hubs, and Application Insights](https://devblogs.microsoft.com/cse/2021/05/13/observability-for-event-stream-processing-with-azure-functions-event-hubs-and-application-insights/#2-understanding-operation-ids-operation-links-when-working-with-event-hubs)

## Related resources

- [Monitor Azure Functions and Event Hubs](/azure/architecture/serverless/event-hubs-functions/observability)
