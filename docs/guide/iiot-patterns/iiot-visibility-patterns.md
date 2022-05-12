Visibility enables manufacturers to gain insights & drive decision-making to improve quality and be more efficient and improve safety. It involves visualizing and correlating IoT data with multiple other business systems including Historians, MES, ERP, QMS to impact metrics like OEE, energy costs, machine downtime, labor efficiency which directly impacts the business value.

Following section includes common visibility patterns for industrial solutions. 

## Time Series Analysis

![Time Series Analysis](images/time-series-analysis.png)

- Dataflow
    1. EdgeHub sends the data to IoT Hub/ Central using AMQP or MQTT.
    1. Data from IoT Hub / Central is pushed to Data Explorer using Data Connection in IoT Hub or Data Export in IoT Central.
    1. Data Explorer dashboards use kql query langauge to fetch the data from the clusters and build near real-time dashboards.
    1. Use Power BI or Grafana to build more custom dashboards with query builder and integration with other data sources.

- Use this pattern when:
    - Need time series analysis for large scale IIoT telemetry data.
    - Need real-time dashboards and querying capabilities on the factory floor.
    - Perform univariate anomaly detection and correlation between sensors.

- Considerations
    - IoT Central includes dashboards for basic time series analysis on last 30 days of data. For analysis on larger datasets beyong 30 days, use Data explorer.
    - Data explorer is an append only platform, [not suitable for data which requires frequent updates.](https://docs.microsoft.com/en-us/azure/data-explorer/data-explorer-overview) 
    - Data explorer
    - [Time Series Analysis in Data Explorer](https://docs.microsoft.com/en-us/azure/data-explorer/time-series-analysis)
    - [Considerations around Streaming Ingestion for Data Explorer](https://docs.microsoft.com/en-us/azure/data-explorer/ingest-data-streaming?tabs=azure-portal%2Ccsharp)
    - [Disaster recovery configurations for Data Explorer](https://docs.microsoft.com/en-us/azure/data-explorer/business-continuity-overview#disaster-recovery-configurations)
    - [Migrating from Time Series Insights (TSI)](https://docs.microsoft.com/en-us/azure/time-series-insights/migration-to-adx)
    
- Deployment Sample
    - [Operational Visibility with Anomaly Detection and Root Cause Analysis](https://github.com/Azure-Samples/industrial-iot-patterns/tree/main/2_OperationalVisibility)


## Anomaly Detection and Root Cause Analysis

![Anomaly Detection and RCA](images/anomaly-detection.png)

- Dataflow
    1. EdgeHub sends the data to IoT Hub/ Central using AMQP or MQTT.
    1. Data from IoT Hub / Central is pushed to Data Explorer using Data Connection in IoT Hub or Data Export in IoT Central.
    1. Data Explorer dashboards use kql query langauge to fetch the data from the clusters and build near real-time dashboards.
    1. Metrics Advisor fetches data from Data Explorer using data feed configuration. It configures the metrics level configuration for anomaly detection and an alert that links to a webhook.
    2. Metrics advisor web hook is connected to a http triggered Logic apps, which gets called when an anomaly is detected.

- Use this pattern when:
    - Need automatic anomaly detection based on machine learning algorithms and range thresholds.
    - Need no code / low code way to build time series machine learning models.
    - Need anomaly incident management and business action alerts.
    - Perform root cause analysis and correlation mapping .

- Considerations
    - [Onboard metric data to Metrics Advisor](https://docs.microsoft.com/en-us/azure/applied-ai-services/metrics-advisor/how-tos/onboard-your-data)
    - [Data feed management for Metrics Advisor](https://docs.microsoft.com/en-us/azure/applied-ai-services/metrics-advisor/how-tos/manage-data-feeds)
    - [Data requirements for Metrics Advisor anomaly detection](https://docs.microsoft.com/en-us/azure/applied-ai-services/metrics-advisor/faq#how-much-data-is-needed-for-metrics-advisor-to-start-anomaly-detection-)
    - [Cost management for Metrics Advisor](https://docs.microsoft.com/en-us/azure/applied-ai-services/metrics-advisor/cost-management#key-points-about-cost-management-and-pricing)
    
- Deployment Sample
    - [Operational Visibility with Anomaly Detection and Root Cause Analysis](https://github.com/Azure-Samples/industrial-iot-patterns/tree/main/2_OperationalVisibility)



# Next steps

- Try the deployment sample for [Operational Visibility with Anomaly Detection and Root Cause Analysis](https://github.com/Azure-Samples/industrial-iot-patterns/tree/main/2_OperationalVisibility)

- [Metrics Advisor Overview](https://docs.microsoft.com/en-us/azure/applied-ai-services/metrics-advisor/overview)

# Related resources

- [Industrial IoT Connectivity Patterns](./iiot-connectivity-patterns.md)

- [Industrial IoT Transparency Patterns](./iiot-transparency-patterns.md)

- [Industrial IoT Prediction Patterns](./iiot-prediction-patterns.md)

- [Solutions for the manufacturing industry](https://docs.microsoft.com/en-us/azure/architecture/industries/manufacturing)

- [IoT Well-Architected Framework](https://docs.microsoft.com/en-us/azure/architecture/framework/iot/iot-overview)