---
title: Reliable web app pattern
description: Learn about the reliable web app pattern.
author: stephen-sumner    
ms.author: ssumner
ms.reviewer: ssumner
ms.date: 04/28/2022
ms.topic: conceptual
ms.service: architecture-center
ms.subservice: azure-guide
azureCategories:
    - web
    - developer-tools
    - databases
    - devops
products:
  - azure
  - azure-app-service
  - azure-cache-redis
categories:
  - web
---

# Reliable web app pattern

The reliable web app pattern provides essential implementation guidance for web apps moving to the cloud. It defines how you should update (re-platform) your web app to be successful in the cloud.

The reliable web app pattern focuses on minimal code changes, reliability design patterns, and managed services so you can rapidly adopt the cloud (*see figure 1*).

[![Diagram showing the principles of the reliable web app pattern](images/reliable-web-app-overview.svg)](images/reliable-web-app-overview.svg#lightbox)
*Figure 1. Overview of the reliable web app pattern.*

The reliable web app pattern builds on the principles of the Azure Well-Architected Framework and highlights several that are essential for the cloud adoption journey. The reliable web app pattern helps ensure web app is cost optimized, observable, and ingress secure. It also improves web app to use infrastructure as code and identity based security.

## Next steps

We created reliable web app pattern guidance for .NET and Java web apps. Both have a reference implementation (sample web app) with the pattern applied that you should deploy. Find the right guidance for your web app and deploy the reference implementation.

>[!div class="nextstepaction"]
>[Reliable web app pattern for .NET](./dotnet/plan-implementation.yml)

>[!div class="nextstepaction"]
>[Reliable web app pattern for Java](./java/plan-implementation.yml)
