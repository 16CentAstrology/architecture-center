This example workload illustrates a Greenfield solution to build a scalable data platform using Microsoft Fabric and the lakehouse design paradigm. Microsoft Fabric is a platform that integrates data storage, processing, and analytics. A Greenfield lakehouse provides a clean slate for designing an efficient, future-proof data ecosystem.

This architecture is applicable to the following scenarios:

- Organizations looking to start fresh, unencumbered by legacy systems, when developing a data platform.
- Organizations that anticipate data volumes between 0.5 TB to 1.5 TB.
- Organizations with a preference for a simple and streamlined pattern that balances cost, complexity, and performance considerations.

## Architecture

![Diagram illustrates a greenfield solution to build a robust, scalable data platform using the lakehouse design paradigm on Microsoft Fabric](media/greenfield-lakehouse-fabric/greenfield-lakehouse-fabric.png)

*Download a [Visio file](media/greenfield-lakehouse-fabric/greenfield-lakehouse-fabric.vsdx) of this architecture.*

### Dataflow

This design reflects the Lambda architecture, which separates data processing into two layers:

1. A high-volume batch processing layer for processed periodically for historical analysis.
2. A low-latency high-throughput stream processing layer for real-time analytics.

The stream processing path ingests and processes data in near real-time, making it ideal for dashboards and anomaly detection. The batch processing path handles the complete dataset, ensuring data consistency and enabling complex historical analysis. This two-pronged approach offers real-time insights while maintaining a reliable record for later exploration.

#### Cold path - Batch analytics

Data warehouses, which relied on relational SQL semantics, were the conventional approach for historical data analysis. However, this pattern has evolved over time, and Lakehouses have emerged as the industry standard for batch data analysis. A Lakehouse is built on top of open file formats and, unlike traditional data warehouses, caters to all types of data - structured, semi-structured, and unstructured. The compute layer in a Lakehouse is typically built on top of the Apache Spark framework, which has become the preferred engine for processing big data due to its distributed computing capability and high performance. Fabric offers a native Lakehouse experience based on the open Delta Lake file format and a managed Spark runtime.

A Lakehouse implementation typically uses [medallion architecture](https://learn.microsoft.com/en-us/azure/databricks/lakehouse/medallion), where the bronze layer contains the raw data, the silver layer contains the validated and deduplicated data, and the gold layer contains highly refined data suitable for supporting business-facing use cases. This approach is agnostic of organizations and industries. While this is the general approach, organizations can customize it for their specific requirements. Let us explore how we can build a Lakehouse using native Fabric components.

##### 1. Data ingestion using Data Factory

[Data factory](https://learn.microsoft.com/en-us/fabric/data-factory/data-factory-overview) experience in Fabric has been built using the capabilities of Azure Data Factory, which is a widely used data integration service in the industry. While Data Factory in Azure mainly provides orchestration capabilities using pipelines, in Fabric, it includes both pipelines and dataflows.

- Data pipelines enable you to apply out-of-the-box rich data orchestration capabilities to compose flexible data workflows that meet your enterprise needs.
- Dataflows enable you to use more than 300 transformations in the dataflows designer, allowing you to transform data using a Power Query-like graphical interface, including smart AI-based data transformations. Dataflows can also write data to native data stores in Fabric, such as Lakehouse, Warehouse, Azure SQL, and Kusto databases.

Depending on your requirements, you can use either or both experiences to build a rich metadata-driven ingestion framework. You can onboard data from various source systems on a defined schedule or based on event triggers.

##### 2. Data transformations

For data preparation and transformation, there are two different approaches. There are Spark notebooks for users who prefer a code-first experience and dataflows for users who prefer a low-code or no-code experience.

[Fabric notebooks](https://learn.microsoft.com/en-us/fabric/data-engineering/how-to-use-notebook) are a primary code item for developing Apache Spark jobs. They provide a web-based interactive surface used by data engineers to write code, benefiting from rich visualizations and Markdown text. Data engineers write code for data ingestion, data preparation, and data transformation. Data scientists also use notebooks to build machine learning solutions, including creating experiments and models, model tracking, and deployment.

Every workspace in Fabric comes with a Spark [starter pool](https://learn.microsoft.com/en-us/fabric/data-engineering/configure-starter-pools), which is used for default Spark jobs. With starter pools, you can expect rapid Apache Spark session initialization, typically within 5 to 10 seconds, with no need for manual setup. You also get the flexibility to customize Apache Spark pools according to your specific data engineering requirements. You can size the nodes, autoscale, and dynamically allocate executors based on your Spark job requirements. For Spark runtime customizations, you have the option to [environments](https://learn.microsoft.com/en-us/fabric/data-engineering/create-and-use-environment). In an environment, you can configure compute properties, select different runtimes, and set up library package dependencies based on your workload requirements.

[Dataflows](https://learn.microsoft.com/en-us/fabric/data-factory/create-first-dataflow-gen2) allow you to extract data from various sources, transform it using a wide range of transformation operations, and load it into a destination. Traditionally, data engineers spend significant time extracting, transforming, and loading data into a consumable format for downstream analytics. The goal of Dataflows Gen2 is to provide an easy, reusable way to perform ETL tasks using visual cues in Power Query Online. Adding a data destination to your dataflow is optional, and the dataflow preserves all transformation steps. To perform other tasks or load data to a different destination after transformation, create a Data Pipeline and add the Dataflow Gen2 activity to your pipeline orchestration.

#### Hot path - Real-time analytics

Real-time data processing is vital for businesses that want to stay agile, make informed decisions quickly, and take advantage of immediate insights to improve operations and customer experiences. In Fabric, this capability is provided by the Real-Time Intelligence service. It comprises several Fabric items bundled together and made discoverable using the [Real-Time Hub](https://learn.microsoft.com/en-us/fabric/real-time-hub/real-time-hub-overview). It's the single place for all data-in-motion across your entire organization.

Real-Time Intelligence in Fabric enables analysis and data visualization for event-driven scenarios, streaming data, and data logs. It connects time-based data from various sources using a catalog of no-code connectors and provides an end-to-end solution for data ingestion, transformation, storage, analytics, visualization, tracking, AI, and real-time actions. Even though it is called "real-time," your data doesn't have to be flowing at high rates and volumes. Real-Time Intelligence gives you event-driven, rather than schedule-driven, solutions.

##### 3. Real time ingestion

[Eventstream](https://learn.microsoft.com/en-us/fabric/real-time-intelligence/event-streams/overview?tabs=enhancedcapabilities) is a Fabric item that enables a no-code way to ingest real-time events from various sources and send them to different destinations. It allows data filtering, transformation, aggregation, and routing based on content. It also enables the creation of new streams from existing ones that can be shared across the organization in the Real-Time Hub. Eventstreams support multiple data sources and data destinations, including a wide range of connectors to external sources, such as Apache Kafka clusters, database change data capture feeds, AWS streaming sources (Kinesis), and Google (GCP Pub/Sub).

You create an eventstream, add event data sources to the stream, optionally add transformations to transform the event data, and then route the data to supported [destinations](https://learn.microsoft.com/en-us/fabric/real-time-intelligence/event-streams/overview?tabs=enhancedcapabilities#route-events-to-destinations). One of the supported destinations is a Fabric Lakehouse. This gives you the ability to transform your real-time events before ingesting them into your lakehouse. Real-time events are converted into Delta Lake format and then stored in the designated lakehouse tables. This pattern enables data warehousing scenarios and historical analysis of your fast-moving data.

##### 4. Real time analytics

Within the Real-Time Intelligence experience in Fabric, depending on your use cases, there are two typical pathways for streaming data: [Reflex](https://learn.microsoft.com/en-us/fabric/data-activator/data-activator-get-started#create-a-reflex-item) items and [Eventhouses](https://learn.microsoft.com/en-us/fabric/real-time-intelligence/eventhouse).

Reflex is a Fabric item that allows you to react to the occurrence of a data condition as it happens. That 'reaction' can be a simple alert message via Email/Teams, or it can involve invoking a custom action by triggering a Power Automate flow. You can also trigger any Fabric item from your reflexes. There are many observability use cases supported by Reflex, one of which is reacting to streaming data as it arrives in Eventstreams.

Eventhouse is a collection of one or more KQL databases. KQL databases are engineered for time-based, streaming events with structured, semi-structured, and unstructured data. Data is automatically indexed and partitioned based on ingestion time, giving you incredibly fast and complex analytic querying capabilities, even as the data is streaming in. Data stored in Eventhouses can be made available in OneLake for consumption by other Fabric experiences. Data stored in Eventhouses can be queried using various code, low-code, or no-code options in Fabric. Data can be queried in native [KQL](https://learn.microsoft.com/en-us/fabric/real-time-intelligence/kusto-query-set?tabs=kql-database)(Kusto Query Language) or using T-SQL in the KQL queryset.

[Real time dashboards](https://learn.microsoft.com/en-us/fabric/real-time-intelligence/dashboard-real-time-create) are designed to provide immediate insights from data streaming into your Eventhouses. You can add various types of visuals to the dashboard, such as charts and graphs, and customize them to fit your needs. RT dashboards serve the specific use case of quickly identifying trends and anomalies in high-velocity data arriving in an Eventhouse. This is different from Power BI dashboards, which are suitable for enterprise BI reporting workloads.

##### 5. Data Serving

Depending on the user persona, there are various low-code or pro-code options available to consume data from Fabric Lakehouses and Eventhouses.

###### SQL analytics endpoint

A [SQL analytics endpoint](https://learn.microsoft.com/en-us/fabric/data-engineering/lakehouse-overview#lakehouse-sql-analytics-endpoint) is automatically generated for every Lakehouse in Microsoft Fabric. The SQL analytics endpoint is read-only, and data can only be modified through the "Lake" view of the Lakehouse using Spark. You can query data using the SQL analytics endpoint directly in the Fabric portal by transitioning from the 'Lake' to the 'SQL' view of the Lakehouse. Alternatively, you can use the 'SQL connection string' of a Lakehouse to connect using client tools like Power BI, Excel, SQL Server Management Studio, etc. This is suitable for data/business analyst personas in a data team.

###### Spark notebooks

Notebooks are a popular way to interact with Lakehouse data. Fabric provides a web-based interactive surface used by data workers to write code, applying rich visualizations and Markdown text. Data engineers write code for data ingestion, data preparation, and data transformation. Data scientists also use notebooks for data exploration, creating ML experiments and models, model tracking, and deployment. This option is suitable for professional data engineers and data scientists.

###### Power BI

Every Lakehouse in Fabric comes with a prebuilt default semantic model. It's automatically created when you set up a Lakehouse and load data into it. These models inherit business logic from the Lakehouse, making it easier to create Power BI reports and dashboards directly within the Lakehouse experience. You can also create custom semantic models on Lakehouse tables based on specific business requirements. When you create Power BI reports on a Lakehouse, you can use the [direct lake mode](https://learn.microsoft.com/en-us/fabric/get-started/direct-lake-overview), which doesn't require you to import data separately. This allows you to achieve in-memory performance on your reports without moving your data out of the Lakehouse.

###### Custom APIs

Fabric has a rich API surface across its different items. Microsoft OneLake provides open access to all of Fabric items through existing ADLS Gen2 APIs and SDKs. You can access your data in OneLake through any API, SDK, or tool compatible with ADLS Gen2 just by using a OneLake URI instead. You can upload data to a Lakehouse through Azure Storage Explorer or read a delta table through a shortcut from Azure Databricks. OneLake also supports the [Azure Blob Filesystem driver](https://learn.microsoft.com/en-us/azure/storage/blobs/data-lake-storage-abfs-driver) (ABFS) for more compatibility with ADLS Gen2 and Azure Blob Storage. For consuming streaming data in downstream apps, you can push Eventstream data to a custom API endpoint. This streaming output from Fabric can then be consumed using Eventhub or AMQP and Kafka protocols.

###### Power Automate

Power Automate Flow is a low-code application platform that enables you to automate repetitive tasks in an enterprise and also 'act' on your data. The Reflex item in Fabric provides Power Automate flows as one of the supported destinations. This [integration](https://learn.microsoft.com/en-us/fabric/data-activator/data-activator-trigger-power-automate-flows) unlocks many use cases and allows you to trigger downstream actions using a wide range of connectors, both Microsoft native and external systems.

### Components

The following components are used to enable this solution:

- [Microsoft Fabric](): An end-to-end cloud-based data analytics platform designed for enterprises that offers a unified environment for various data tasks like data ingestion, transformation, analysis, and visualization.

  - [OneLake](https://learn.microsoft.com/fabric/onelake/onelake-overview): The central hub for all your data within Microsoft Fabric. It's designed as an open data lake, meaning it can store data in its native format regardless of structure.

  - [Data Factory](https://learn.microsoft.com/fabric/data-factory/data-factory-overview): A cloud-based ETL and orchestration service for automated data movement and transformation. It allows you to automate data movement and transformation at scale across various data sources.

  - [Data Engineering](https://learn.microsoft.com/fabric/data-engineering/data-engineering-overview): Tools that enable the collection, storage, processing, and analysis of large volumes of data.

  - [Data Science](https://learn.microsoft.com/fabric/data-science/data-science-overview): Tools that empower you to complete end-to-end data science workflows in data enrichment and business insights.

  - [Real-Time Intelligence](https://learn.microsoft.com/fabric/real-time-intelligence/overview): Provides stream ingestion and processing capabilities. This allows you to gain insights from constantly flowing data, enabling quicker decision-making based on real-time trends and anomalies.

  - [Power BI](https://learn.microsoft.com/power-bi/fundamentals/power-bi-overview): Business intelligence tool for creating interactive dashboards and reports to visualize data and gain insights.

  - [Copilot](https://learn.microsoft.com/fabric/get-started/copilot-fabric-overview): Allows you to analyze data, generate insights, and create visualizations and reports in Microsoft Fabric and Power BI using natural language.

### Alternatives

Microsoft Fabric offers a robust set of tools, but depending on your specific needs, alternative services within the Azure ecosystem can be used for enhanced functionality.

- [Azure Databricks](https://learn.microsoft.com/azure/databricks/introduction/) could replace or complement the Microsoft Fabric native Data Engineering capabilities. Azure Databricks offers an alternative for large-scale data processing by providing a cloud-based Apache Spark environment. Azure Databricks also extends this by providing common governance across your entire data estate, and capabilities to enable key use cases including data science, data engineering, machine learning, AI, and SQL-based analytics.

- [Azure Machine Learning](https://learn.microsoft.com/azure/machine-learning/overview-what-is-azure-machine-learning) could replace or complement the Microsoft Fabric native Data Science. Azure Machine Learning goes beyond the model experimentation and management capabilities in Microsoft Fabric by adding capabilities to allow you to host models for online inference use-cases, monitor models for drift, and provide capabilities to build custom Generative-AI applications.

## Scenario details

Several scenarios can benefit from this workload:

- Organizations starting fresh without legacy system constraints.
- Businesses needing a simple, cost-effective, and high-performance data platform that addresses reporting, analytics, and machine learning requirements.
- Organizations looking to integrate data from multiple sources for a unified view.

This solution isn't recommended for:

- Teams from a SQL or relational database background with limited skills on Apache Spark.
- Organizations who are migrating from a legacy system or data warehouse to a modern platform.

## Considerations

These considerations implement the pillars of the Azure Well-Architected Framework, which is a set of guiding tenets that can be used to improve the quality of a workload. For more information, see [Microsoft Azure Well-Architected Framework](https://learn.microsoft.com/azure/well-architected/).

The following considerations apply to this scenario.

### Reliability

Fabric automatically replicates resources across availability zones without any need for you to set up or configure. For example, during a zone-wide outage, no action is required to recover a zone. In [supported regions](https://learn.microsoft.com/en-us/azure/reliability/reliability-fabric#supported-regions), Fabric can self-heal and rebalance automatically to take advantage of the healthy zone.

### Security

Fabric also allows you to manage, control and audit your security settings, in line with your changing needs and demands. Key security considerations in Fabric include: 

- Authentication: Configure single sign-on (SSO) in Microsoft Entra ID for access across various devices and locations.

- Role-Based Access Control (RBAC): Implement workspace-based access control to precisely manage who can access and interact with specific datasets, ensuring users only access what they are authorized to.

- Network Security: Utilize Fabric’s inbound and outbound network security controls when connecting to data or services within or outside your network. Key features include: [Conditional Access](https://learn.microsoft.com/en-us/fabric/security/security-conditional-access), [Private Links](https://learn.microsoft.com/en-us/fabric/security/security-private-links-overview), [Trusted Workspace Access](https://learn.microsoft.com/en-us/fabric/security/security-trusted-workspace-access), [Managed Private Endpoints](https://learn.microsoft.com/en-us/fabric/security/security-managed-private-endpoints-overview), and more.

- Audit Logs: Use Fabric’s detailed audit logs to track user activities and ensure accountability across the platform.

For more information, see [Security in Microsoft Fabric](https://learn.microsoft.com/en-us/fabric/security/security-overview). 

### Cost optimization

Cost optimization is about looking at ways to reduce unnecessary expenses and improve operational efficiencies. For more information, see [Overview of the cost optimization pillar](/azure/architecture/framework/cost/overview).

Microsoft Fabric offers [capacity reservations](https://learn.microsoft.com/en-us/azure/cost-management-billing/reservations/fabric-capacity) for a given number of capacity units (CUs). Capacity reservations can help you save costs by committing to a reservation for your Fabric capacity usage for a duration of one year.

To maximise the utilisation of your capacity Microsoft Fabric connsider the following:

1. Right sizing F SKU - To help you determine the right capacity size, you can provision [trial capacities](https://learn.microsoft.com/en-us/fabric/get-started/fabric-trial) or [pay-as-you-go F SKUs](https://learn.microsoft.com/en-us/fabric/enterprise/buy-subscription#azure-skus) to measure the actual capacity size required before purchasing an F SKU reserved instance. It is recommended that you perform a scoped proof of concept with a representative workload, monitor the CU usage, and then extrapolate to arrive at an estimated CU usage in production. Fabric allows for seamless scaling. You can start with a conservative capacity size and scale up as you need more capacity.
 
2. Monitor Usage Patterns: Regularly track and analyze your usage to identify peak and off-peak hours. This helps in understanding when your resources are most utilized and can guide you in scheduling non-critical tasks during off-peak times to avoid spikes in CU usage.
 
3. Optimize Queries and Workloads: Ensure that your queries and workloads are optimized to reduce unnecessary compute usage. This includes optimizing DAX queries, Python code, and other operations to be more efficient.
 
4. Use Capacity Reservations: Consider committing to a capacity reservation for a year. This can provide significant cost savings compared to pay-as-you-go models. See details [here](https://learn.microsoft.com/en-us/azure/cost-management-billing/reservations/fabric-capacity).
 
5. Leverage Bursting and Smoothing: Utilize Fabric’s bursting and smoothing features to handle CPU-intensive activities without needing a higher SKU. This helps in managing costs while maintaining performance. See details [here](https://learn.microsoft.com/en-us/fabric/enterprise/optimize-capacity).
 
6. Set Up Alerts and Notifications: Configure proactive alerts for capacity admins to monitor and manage high compute usage. This can help in taking timely actions to prevent cost overruns.
 
7. Implement workload management: Schedule log-running jobs at staggered times based on resource availability and system demand to optimize capacity usage. See more information [here](https://learn.microsoft.com/en-us/fabric/data-warehouse/workload-management).

Other cost optimization considerations include:

- [Data Lake Storage Gen2](https://azure.microsoft.com/pricing/details/storage/data-lake/) pricing depends on the amount of data you store and how often you use the data. The sample pricing includes 1 TB of data stored, with further transactional assumptions. The 1 TB refers to the size of the data lake, not the original legacy database size.
- [Microsoft Fabric](https://azure.microsoft.com/en-us/pricing/details/microsoft-fabric/) pricing is based on Fabric F capacity price or Premium Per Person price. Serverless capacities would consume CPU and memory from dedicated capacity that was purchased.
- [Event Hubs](https://azure.microsoft.com/pricing/details/event-hubs/) bills based on tier, throughput units provisioned, and ingress traffic received. The example assumes one throughput unit in Standard tier over one million events for a month.

### Operational excellence

Microsoft Fabric provides many different components to help you manage your data platform. Each of these experiences supports unique operations that can be viewed in the [Microsoft Fabric Capacity Metrics app](https://learn.microsoft.com/en-us/fabric/enterprise/metrics-app). Use the Microsoft Fabric Capacity Metrics app to monitor your capacity consumption and make informed decisions on how to use your capacity resources.

### Performance efficiency

Microsoft Fabric provides several features to optimize performance across its components. These tools help manage compute resources effectively, prevent overloads, and guide scaling decisions. These tools and practices help you manage compute resources effectively, prevent overloading, and make informed decisions on scaling and optimizing workloads.

Some key performance efficiency capabilities in Fabric include:

- Use [bursting and smoothing](https://blog.fabric.microsoft.com/blog/fabric-capacities-everything-you-need-to-know-about-whats-new-and-whats-coming?ft=All#BurstSmooth) to ensure CPU-intensive activities are completed quickly without requiring a higher SKU. Schedule these activities at any time of the day.

- Use [throttling](https://learn.microsoft.com/en-us/fabric/enterprise/throttling) to delay or reject operations when capacity experiences sustained high CPU demand (above the SKU limit). More details here.

- Use the [Fabric Capacity Metrics App](https://learn.microsoft.com/en-us/fabric/enterprise/metrics-app) to visualize capacity usage, optimize performance of artifacts, and optimize high-compute items. It differentiates between interactive operations (like DAX queries) and background operations (like semantic model refreshes) for targeted optimizations.

## Contributors

_This article is being updated and maintained by Microsoft. It was originally written by the following contributors._

Principal author:

- [Amit Chandra](https://www.linkedin.com/in/amitchandra2005/) | Cloud Solution Architect
- [Nicholas Moore](https://www.linkedin.com/in/nicholas-moore/) | Cloud Solution Architect

To see non-public LinkedIn profiles, sign in to LinkedIn.

## Next steps

Consult the relevant documentation to learn more about the different components and how to get started.

- [What is OneLake?](https://learn.microsoft.com/fabric/onelake/onelake-overview)
- [What is Data Factory?](https://learn.microsoft.com/fabric/data-factory/data-factory-overview)
- [What is Data Engineering?](https://learn.microsoft.com/fabric/data-engineering/data-engineering-overview)
- [What is Data science?](https://learn.microsoft.com/fabric/data-science/data-science-overview)
- [What is Real-Time Intelligence?](https://learn.microsoft.com/fabric/real-time-intelligence/overview)
- [What is Power BI?](https://learn.microsoft.com/power-bi/fundamentals/power-bi-overview)
- [Introduction to Copilot in Fabric](https://learn.microsoft.com/fabric/get-started/copilot-fabric-overview)

## Related resources

- Learn more about:
  - [Data lakes](../../data-guide/scenarios/data-lake.md)
  - [Data warehousing and analytics](data-warehouse.yml)
  - [Enterprise business intelligence](/azure/architecture/example-scenario/analytics/enterprise-bi-synapse)
