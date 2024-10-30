---
author: ssumner
ms.author: ssumner
ms.date: 10/15/2024
ms.topic: include
ms.service: azure-architecture-center
---
Autoscaling ensures that a web app remains resilient, responsive, and capable of handling dynamic workloads efficiently. To implement autoscaling, follow these recommendations:

- *Automate scale-out.* Use [Azure autoscale](/azure/azure-monitor/autoscale/autoscale-overview) to automate horizontal scaling in production environments. Configure autoscaling rules to scale out based on key performance metrics, so your application can handle varying loads.

- *Refine scaling triggers.* Begin with CPU utilization as your initial scaling trigger if you're unfamiliar with your application’s scaling requirements. Refine your scaling triggers to include other metrics such as RAM, network throughput, and disk I/O. The goal is to match your web application's behavior for better performance.

- *Provide a scale-out buffer.* Set your scaling thresholds to trigger before reaching maximum capacity. For example, configure scaling to occur at 85% CPU utilization rather than waiting until it reaches 100%. This proactive approach helps maintain performance and avoid potential bottlenecks.