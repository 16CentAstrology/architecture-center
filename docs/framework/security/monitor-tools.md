---
title: Azure security monitoring tools
description: Tools that provide observability into threats and vulnerable resources used by the workload.
author: v-aangie
ms.date: 09/20/2020
ms.topic: conceptual
ms.service: architecture-center
ms.subservice: well-architected
products:
  - azure-security-center
  - azure-sentinel
  - m365-security-center
subject: 
  - security
  - monitor
ms.custom:
  - article
---

# Azure security monitoring tools

The _leverage native control_ security principle tells us to use native controls built into cloud services over external controls through third-party solutions. Native reduce the effort required to integrate external security tooling and update those integrations over time.

Azure provides several monitoring tools that observe the operations and detect anomalous behavior. These tools can detect threats at different levels and report issues. Addressing the issues early in the operational lifecycle will strengthen your overall security posture.

## Tools

|Service|Use case|
|---|---|
|[**Azure Security Center**](/azure/security-center/security-center-intro)| Strengthens the security posture of your data centers, and provides advanced threat protection across your hybrid workloads in the cloud (whether they're in Azure or not) and  on-premises. Get a unified view into the infrastructure and resources provisioned for the workload. |
|[**Azure Sentinel**](/azure/sentinel/overview)|Use the native security information event management (SIEM) and security orchestration automated response (SOAR) solution on Azure. Receive intelligent security analytics and threat intelligence across the enterprise.|
|[**Azure DDoS Protection**](/azure/virtual-network/ddos-protection-overview)| Defend against distributed denial of service (DDoS) attacks.|
|[**Azure Rights Management (RMS)**](/azure/information-protection/what-is-azure-rms)| Protect files and emails across multiple devices.|
|[**Microsoft Information Protection (MIP)**](/information-protection/develop/overview)| Secure email, documents, and sensitive data that you share outside your company.|

## Azure Security Center

Enable Azure Security Center at the subscription level to monitor all resource provisioned in that scope. At no additional cost, it provides continuous observability into resources, reports issues, and recommends fixes. By regularly reviewing and fixing issues, you can improve the security posture, detect threats early, prevent breaches.

Beyond just observability, Security Center offers an advanced mode through its integration with **Azure Defender**. When this plan is enabled, built-in policies, custom policies, and initiatives protect resources and block malicious actors. You can also monitor compliance with regulatory standards - such as NIST, Azure CIS, Azure Security Benchmark. For pricing details, see [Security Center pricing](https://azure.microsoft.com/en-us/pricing/details/azure-defender/). 


## Azure Sentinel

If your business has hybrid workloads that runs on and multiple cloud platforms, and, or across cloud and on-premises, make sure that you have a centralized view of all data related to those resources. To get that view you need security information event management (SIEM) and security orchestration automated response (SOAR) solutions. These solutions connect to all security sources, monitor them, and analyze the correlated data.

Azure Sentinel and is a native control that combines SIEM and SOAR capabilities. It collects events and logs from various connected sources. Based on the data sources and their alerts, Sentinel creates incidents, performs threat analysis for early detection. Through intelligent analytics and queries, you can be proactive with hunting activities. In case of incidents, you can automate workflows. Also, with workbook templates you can quickly gain insights through visualization.

## Azure DDoS Protection

A Distributed Denial of Service (DDoS) attack attempts to exhaust an application's resources, making the application unavailable to legitimate users. DDoS attacks can be targeted at any endpoint that is publicly reachable through the internet.

Every property in Azure is protected by Azure's infrastructure DDoS (Basic) Protection at no additional cost. The scale and capacity of the globally deployed Azure network provides defense against common network-layer attacks through always-on traffic monitoring and real-time mitigation. DDoS Protection Basic requires no user configuration or application changes. DDoS Protection Basic helps protect all Azure services, including PaaS services like Azure DNS.

Azure DDoS Protection Standard, combined with application design best practices, provides enhanced DDoS mitigation features to defend against DDoS attacks. It is automatically tuned to help protect your specific Azure resources in a virtual network. Protection is simple to enable on any new or existing virtual network, and it requires no application or resource changes. It has several advantages over the basic service, including logging, alerting, telemetry, SLA guarantee and cost protection.

Azure DDoS Protection Standard is designed for [services that are deployed in a virtual network](https://docs.microsoft.com/azure/virtual-network/virtual-network-for-azure-services). For other services, the default DDoS Protection Basic service applies. To learn more about supported architectures, see [DDoS Protection reference architectures](https://docs.microsoft.com/azure/ddos-protection/ddos-protection-reference-architectures).

## Azure Rights Management (RMS)

Your business may encounter challenges with protecting documents and emails. For example, file protection, collaboration, and sharing may be issues. You also might be experiencing problems regarding platform support or infrastructure.

[Azure Rights Management (RMS)](/azure/information-protection/what-is-azure-rms) is a cloud-based protection service that uses encryption, identity, and authorization policies to help secure files and emails across multiple devices, including phones, tablets, and PCs.

To learn more about how RMS can address these issues, see [Business problems solved by Azure Rights Management](/azure/information-protection/what-is-azure-rms#business-problems-solved-by-azure-rights-management).

## Microsoft Information Protection (MIP)

The *data classification* process categorizes data by sensitivity and business impact in order to identify risks. When data is classified, you can manage it in ways that protect sensitive or important data from theft or loss.

With proper *file protection*, you can analyze data flows to gain insight into your business, detect risky behaviors and take corrective measures, track access to documents, and more. The protection technology in AIP uses encryption, identity, and authorization policies. Protection stays with the documents and emails, independently of the location, regardless of whether they are inside or outside your organization, networks, file servers, and applications

[Azure Information Protection (AIP)](/azure/information-protection/what-is-information-protection) is part of Microsoft Information Protection (MIP) solution, and extends the labeling and classification functionality provided by Microsoft 365. For more information, see [Sensitivity labels](microsoft-365/compliance/sensitivity-labels) and 
[classification](/microsoft-365/compliance/data-classification-overview).


## Next steps
> [!div class="nextstepaction"]
> [Monitor workload resources in Azure Security Center](monitor-resources.md)

## Related links

For information on the Azure Security Center tools, see [Strengthen security posture](/azure/security-center/security-center-intro#strengthen-security-posture).

For frequently asked questions on Azure Security Center, see [FAQ - General Questions](/azure/security-center/faq-general).

For information on the Azure Sentinel tools that will help to meet these requirements, see [What is Azure Sentinel?](/azure/sentinel/overview#analytics)

For types of DDoS attacks that DDoS Protection Standard mitigates as well as more features, see [Azure DDoS Protection Standard overview](/azure/virtual-network/ddos-protection-overview).

