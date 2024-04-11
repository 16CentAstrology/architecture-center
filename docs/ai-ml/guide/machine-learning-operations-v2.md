---
title: Machine learning operations (MLOps) v2
description: Learn about a single  deployable set of repeatable, and maintainable patterns for creating machine learning CI/CD and retraining pipelines.
author: sdonohoo
ms.author: robbag
ms.date: 08/04/2022
ms.topic: conceptual
ms.service: architecture-center
ms.subservice: azure-guide
products:
  - azure-machine-learning
  - azure-pipelines
  - azure-arc
  - azure-synapse-analytics
  - azure-event-hubs
categories:
  - ai-machine-learning
---

# Machine learning operations (MLOps) v2

This article describes three Azure architectures for machine learning operations. They all have end-to-end continuous integration (CI), continuous delivery (CD), and retraining pipelines. The architectures are for these AI applications:

- Classical machine learning
- Computer vision (CV)
- Natural language processing (NLP)

The architectures are the product of the MLOps v2 project. They incorporate the best practices that the solution architects discovered in the process of creating multiple machine learning solutions. The result is deployable, repeatable, and maintainable patterns as described here.

All of the architectures use the Azure Machine Learning service.

For an implementation with sample deployment templates for MLOps v2, see [Azure MLOps (v2) solution accelerator](https://github.com/Azure/mlops-v2) on GitHub.

## Potential use cases

- Classical machine learning: Time-Series forecasting, regression, and classification on tabular structured data are the most common use cases in this category. Examples are:
  - Binary and multi-label classification
  - Linear, polynomial, ridge, lasso, quantile, and Bayesian regression
  - ARIMA, autoregressive (AR), SARIMA, VAR, SES, LSTM
- CV: The MLOps framework presented here focuses mostly on the CV use cases of segmentation and image classification.
- NLP: This MLOps framework can implement any of those use cases, and others not listed:
  - Named entity recognition
  - Text classification
  - Text generation
  - Sentiment analysis
  - Translation
  - Question answering
  - Summarization
  - Sentence detection
  - Language detection
  - Part-of-speech tagging

 Simulations, deep reinforcement learning, and other forms of AI aren't covered by this article.

## Architecture

The MLOps v2 architectural pattern is made up of four main modular elements that represent these phases of the MLOps lifecycle:

- Data estate
- Administration and setup
- Model development (inner loop)
- Model deployment (outer loop)

These elements, the relationships between them, and the personas typically associated with them are common for all MLOps v2 scenario architectures. There can be variations in the details of each, depending on the scenario.

The base architecture for MLOps v2 for Machine Learning is the classical machine learning scenario on tabular data. The CV and NLP architectures build on and modify this base architecture.

### Current architectures

The architectures currently covered by MLOps v2 and discussed in this article are:

- [Classical machine learning architecture](#classical-machine-learning-architecture)
- [Machine Learning CV architecture](#machine-learning-cv-architecture)
- [Machine Learning NLP architecture](#machine-learning-nlp-architecture)

### Classical machine learning architecture

:::image type="content" source="_images/classical-ml-architecture.png" lightbox="_images/classical-ml-architecture.png" alt-text="Diagram for the classical machine learning architecture." border="false":::

*Download a [Visio file](https://arch-center.azureedge.net/machine-learning-operation-classical-ml.vsdx) of this architecture.*

#### Workflow for the classical machine learning architecture

1. Data estate

   This element illustrates the data estate of the organization, and potential data sources and targets for a data science project. Data engineers are the primary owners of this element of the MLOps v2 lifecycle. The Azure data platforms in this diagram are neither exhaustive nor prescriptive. The data sources and targets that represent recommended best practices based on the customer use case are indicated by a green check mark.
   
1. Administration and setup

   This element is the first step in the MLOps v2 accelerator deployment. It consists of all tasks related to creation and management of resources and roles associated with the project. These can include the following tasks, and perhaps others:
   1. Creation of project source code repositories
   1. Creation of Machine Learning workspaces by using Bicep, ARM, or Terraform
   1. Creation or modification of datasets and compute resources that are used for model development and deployment
   1. Definition of project team users, their roles, and access controls to other resources
   1. Creation of CI/CD pipelines
   1. Creation of monitors for collection and notification of model and infrastructure metrics

   The primary persona associated with this phase is the infrastructure team, but there can also be data engineers, machine learning engineers, and data scientists.
   
1. Model development (inner loop)

   The inner loop element consists of your iterative data science workflow that acts within a dedicated, secure Machine Learning workspace. A typical workflow is illustrated in the diagram. It proceeds from data ingestion, exploratory data analysis, experimentation, model development and evaluation, to registration of a candidate model for production. This modular element as implemented in the MLOps v2 accelerator is agnostic and adaptable to the process your data science team uses to develop models.

   Personas associated with this phase include data scientists and machine learning engineers.
   
1. Machine Learning registries

   After the data science team develops a model that's a candidate for deploying to production, the model can be registered in the Machine Learning workspace registry. CI pipelines that are triggered, either automatically by model registration or by gated human-in-the-loop approval, promote the model and any other model dependencies to the model deployment phase.

   Personas associated with this stage are typically machine learning engineers.

1. Model deployment (outer loop)

   The model deployment or outer loop phase consists of pre-production staging and testing, production deployment, and monitoring of model, data, and infrastructure. CD pipelines manage the promotion of the model and related assets through production, monitoring, and potential retraining, as criteria that are appropriate to your organization and use case are satisfied.

   Personas associated with this phase are primarily machine learning engineers.
   
1. Staging and test

   The staging and test phase can vary with customer practices but typically includes operations such as retraining and testing of the model candidate on production data, test deployments for endpoint performance, data quality checks, unit testing, and responsible AI checks for model and data bias. This phase takes place in one or more dedicated, secure Machine Learning workspaces.
   
1. Production deployment

   After a model passes the staging and test phase, it can be promoted to production by using a human-in-the-loop gated approval. Model deployment options include a managed batch endpoint for batch scenarios or, for online, near-real-time scenarios, either a managed online endpoint or Kubernetes deployment by using Azure Arc. Production typically takes place in one or more dedicated, secure Machine Learning workspaces.
   
1. Monitoring

   Monitoring in staging, test, and production makes it possible for you to collect metrics for, and act on, changes in performance of the model, data, and infrastructure. Model and data monitoring can include checking for model and data drift, model performance on new data, and responsible AI issues. Infrastructure monitoring can watch for slow endpoint response, inadequate compute capacity, or network problems.
   
1. Data and model monitoring: events and actions

   Based on criteria for model and data matters of concern such as metric thresholds or schedules, automated triggers and notifications can implement appropriate actions to take. This can be regularly scheduled automated retraining of the model on newer production data and a loopback to staging and test for pre-production evaluation. Or, it can be due to triggers on model or data issues that require a loopback to the model development phase where data scientists can investigate and potentially develop a new model.
   
1. Infrastructure monitoring: events and actions

   Based on criteria for infrastructure matters of concern such as endpoint response lag or insufficient compute for the deployment, automated triggers and notifications can implement appropriate actions to take. They trigger a loopback to the setup and administration phase where the infrastructure team can investigate and potentially reconfigure the compute and network resources.

### Machine Learning CV architecture

:::image type="content" source="_images/computer-vision-architecture.png" lightbox="_images/computer-vision-architecture.png" alt-text="Diagram for the computer vision architecture." border="false":::

*Download a [Visio file](https://arch-center.azureedge.net/machine-learning-operation-computer-vision.vsdx) of this architecture.*

#### Workflow for the CV architecture

The Machine Learning CV architecture is based on the classical machine learning architecture, but it has modifications that are particular to supervised CV scenarios.

1. Data estate

   This element illustrates the data estate of the organization and potential data sources and targets for a data science project. Data engineers are the primary owners of this element of the MLOps v2 lifecycle. The Azure data platforms in this diagram are neither exhaustive nor prescriptive. Images for CV scenarios can come from many different data sources. For efficiency when developing and deploying CV models with Machine Learning, recommended Azure data sources for images are Azure Blob Storage and Azure Data Lake Storage.
   
1. Administration and setup

   This element is the first step in the MLOps v2 accelerator deployment. It consists of all tasks related to creation and management of resources and roles associated with the project. For CV scenarios, administration and setup of the MLOps v2 environment is largely the same as for classical machine learning, but with an additional step: create image labeling and annotation projects by using the labeling feature of Machine Learning or another tool.
   
1. Model development (inner loop)

   The inner loop element consists of your iterative data science workflow performed within a dedicated, secure Machine Learning workspace. The primary difference between this workflow and the classical machine learning scenario is that image labeling and annotation is a key element of this development loop.
   
1. Machine Learning registries

   After the data science team develops a model that's a candidate for deploying to production, the model can be registered in the Machine Learning workspace registry. CI pipelines that are triggered either automatically by model registration or by gated human-in-the-loop approval promote the model and any other model dependencies to the model deployment phase.
   
1. Model deployment (outer loop)

   The model deployment or outer loop phase consists of pre-production staging and testing, production deployment, and monitoring of model, data, and infrastructure. CD pipelines manage the promotion of the model and related assets through production, monitoring, and potential retraining as criteria appropriate to your organization and use case are satisfied.
   
1. Staging and test

   The staging and test phase can vary with customer practices but typically includes operations such as test deployments for endpoint performance, data quality checks, unit testing, and responsible AI checks for model and data bias. For CV scenarios, retraining of the model candidate on production data can be omitted due to resource and time constraints. Instead, the data science team can use production data for model development, and the candidate model that's registered from the development loop is the model that's evaluated for production. This phase takes place in one or more dedicated, secure Machine Learning workspaces.
   
1. Production deployment

   After a model passes the staging and test phase, it can be promoted to production via human-in-the-loop gated approvals. Model deployment options include a managed batch endpoint for batch scenarios or, for online, near-real-time scenarios, either a managed online endpoint or Kubernetes deployment by using Azure Arc. Production typically takes place in one or more dedicated, secure Machine Learning workspaces.
   
1. Monitoring

   Monitoring in staging, test, and production makes it possible for you to collect metrics for, and act on, changes in the performance of the model, data, and infrastructure. Model and data monitoring can include checking for model performance on new images. Infrastructure monitoring can watch for slow endpoint response, inadequate compute capacity, or network problems.
   
1. Data and model monitoring: events and actions

   The data and model monitoring and event and action phases of MLOps for NLP are the key differences from classical machine learning. Automated retraining is typically not done in CV scenarios when model performance degradation on new images is detected. In this case, new images for which the model performs poorly must be reviewed and annotated by a human-in-the-loop process, and often the next action goes back to the model development loop for updating the model with the new images.
   
1. Infrastructure monitoring: events and actions

   Based on criteria for infrastructure matters of concern such as endpoint response lag or insufficient compute for the deployment, automated triggers and notifications can implement appropriate actions to take. This triggers a loopback to the setup and administration phase where the infrastructure team can investigate and potentially reconfigure environment, compute, and network resources.

### Machine Learning NLP architecture

:::image type="content" source="_images/natural-language-processing-architecture.png" lightbox="_images/natural-language-processing-architecture.png" alt-text="Diagram for the N L P architecture." border="false":::

*Download a [Visio file](https://arch-center.azureedge.net/machine-learning-operation-natural-language-processing.vsdx) of this architecture.*

#### Workflow for the NLP architecture

The Machine Learning NLP architecture is based on the classical machine learning architecture, but it has some modifications that are particular to NLP scenarios.

1. Data estate

   This element illustrates the organization data estate and potential data sources and targets for a data science project. Data engineers are the primary owners of this element of the MLOps v2 lifecycle. The Azure data platforms in this diagram are neither exhaustive nor prescriptive. Data sources and targets that represent recommended best practices based on the customer use case are indicated by a green check mark.
1. Administration and setup

   This element is the first step in the MLOps v2 accelerator deployment. It consists of all tasks related to creation and management of resources and roles associated with the project. For NLP scenarios, administration and setup of the MLOps v2 environment is largely the same as for classical machine learning, but with an additional step: create image labeling and annotation projects by using the labeling feature of Machine Learning or another tool.
1. Model development (inner loop)

   The inner loop element consists of your iterative data science workflow performed within a dedicated, secure Machine Learning workspace. The typical NLP model development loop can be significantly different from the classical machine learning scenario in that annotators for sentences and tokenization, normalization, and embeddings for text data are the typical development steps for this scenario.
1. Machine Learning registries

   After the data science team develops a model that's a candidate for deploying to production, the model can be registered in the Machine Learning workspace registry. CI pipelines that are triggered either automatically by model registration or by gated human-in-the-loop approval promote the model and any other model dependencies to the model deployment phase.
1. Model deployment (outer loop)

   The model deployment or outer loop phase consists of pre-production staging and testing, production deployment, and monitoring of the model, data, and infrastructure. CD pipelines manage the promotion of the model and related assets through production, monitoring, and potential retraining, as criteria for your organization and use case are satisfied.
1. Staging and test

   The staging and test phase can vary with customer practices, but typically includes operations such as retraining and testing of the model candidate on production data, test deployments for endpoint performance, data quality checks, unit testing, and responsible AI checks for model and data bias. This phase takes place in one or more dedicated, secure Machine Learning workspaces.
1. Production deployment

   After a model passes the staging and test phase, it can be promoted to production by a human-in-the-loop gated approval. Model deployment options include a managed batch endpoint for batch scenarios or, for online, near-real-time scenarios, either a managed online endpoint or Kubernetes deployment by using Azure Arc. Production typically takes place in one or more dedicated, secure Machine Learning workspaces.
1. Monitoring

   Monitoring in staging, test, and production makes it possible for you to collect and act on changes in performance of the model, data, and infrastructure. Model and data monitoring can include checking for model and data drift, model performance on new text data, and responsible AI issues. Infrastructure monitoring can watch for issues such as slow endpoint response, inadequate compute capacity, and network problems.
1. Data and model monitoring: events and actions

   As with the CV architecture, the data and model monitoring and event and action phases of MLOps for NLP are the key differences from classical machine learning. Automated retraining isn't typically done in NLP scenarios when model performance degradation on new text is detected. In this case, new text data for which the model performs poorly must be reviewed and annotated by a human-in-the-loop process. Often the next action is to go back to the model development loop to update the model with the new text data.
1. Infrastructure monitoring: events and actions

   Based on criteria for infrastructure matters of concern such as endpoint response lag or insufficient compute for the deployment, automated triggers and notifications can implement appropriate actions to take. They trigger a loopback to the setup and administration phase where the infrastructure team can investigate and potentially reconfigure the compute and network resources.

### Components

- [Machine Learning](https://azure.microsoft.com/services/machine-learning): A cloud service for training, scoring, deploying, and managing machine learning models at scale.
- [Azure Pipelines](https://azure.microsoft.com/services/devops/pipelines): This build and test system is based on Azure DevOps and is used for the build and release pipelines. Azure Pipelines splits these pipelines into logical steps called tasks.
- [GitHub](https://github.com): A code hosting platform for version control, collaboration, and CI/CD workflows.
- [Azure Arc](https://azure.microsoft.com/services/azure-arc): A platform for managing Azure and on-premises resources by using Azure Resource Manager. The resources can include virtual machines, Kubernetes clusters, and databases.
- [Kubernetes](https://kubernetes.io): An open-source system for automating deployment, scaling, and management of containerized applications.
- [Azure Data Lake](https://azure.microsoft.com/services/storage/data-lake-storage): A Hadoop-compatible file system. It has an integrated hierarchical namespace and the massive scale and economy of Blob Storage.
- [Azure Synapse Analytics](https://azure.microsoft.com/services/synapse-analytics): A limitless analytics service that brings together data integration, enterprise data warehousing, and big data analytics.
- [Azure Event Hubs](https://azure.microsoft.com/services/event-hubs). A service that ingests data streams generated by client applications. It then ingests and stores streaming data, preserving the sequence of events received. Consumers can connect to the hub endpoints to retrieve messages for processing. Here we are taking advantage of the integration with Data Lake Storage.

## Additional considerations

### Persona-based RBAC

When designing solutions using  Azure Machine Learning, managing access to resources is crucial. Role-Based Access Control (RBAC) provides a robust framework for controlling who can perform specific actions and access particular areas within an Azure workspace.

When managing the lifecycle of machine learning models in Azure Machine Learning, it's important to consider the personas involved in the process. Each persona has a specific role and set of responsibilities.

This section aims to outline an exmaple of how to map personas to roles in Azure Machine Learning. The roles are based on the personas involved in the MLOps v2 lifecycle.

#### Example Personas

This example considers the following Personas to inform the identity-based RBAC group design:

##### 1 - Data Scientist/ML Engineer
&nbsp;&nbsp; **Description** - The people doing the various ML and data science activities across the SLDC lifecycle for a project. This role's responsibilies include break and fix activities for the ML models, packages, and data, which sit outside of [platform support](secure_aml_security.md#7---platform-technical-support) expertise.  
<br/>&nbsp;&nbsp; **Type** - Person.<br/>
&nbsp;&nbsp; **Project Specific** - Yes. 
<br/>&nbsp;&nbsp; **Notes** - Involves data exploration and preprocessing to model training, evaluation, and deployment, to solve complex business problems and generate insight.<br/>

##### 2 - Data Analyst
&nbsp;&nbsp; **Description** - The people doing the data analyst tasks required as an input to data science activities. 
<br/>&nbsp;&nbsp; **Type** - Person.<br/>
&nbsp;&nbsp; **Project Specific** - Yes. 
<br/>&nbsp;&nbsp; **Notes** - This role involves working with data, performing analysis, and supporting model development and deployment activities.<br/>

##### R3 - Model Tester
&nbsp;&nbsp; **Description** - The compute process used in Staging & QA testing.
<br/>&nbsp;&nbsp; **Type** - Person.<br/>
&nbsp;&nbsp; **Project Specific** - Yes. 
<br/>&nbsp;&nbsp; **Notes** - This role provides functional segregation from the CI/CD processes.<br/>

##### 4 - Business Stakeholders
&nbsp;&nbsp; **Description** - Business stakeholders attached to the project.
<br/>&nbsp;&nbsp; **Type** - Person.<br/>
&nbsp;&nbsp; **Project Specific** - Yes. 
<br/>&nbsp;&nbsp; **Notes** - This role is [read-only](/azure/role-based-access-control/built-in-roles/general#reader) for the AML workspace components in development.<br/>

##### 5 - Project Lead (Data Science Lead)
&nbsp;&nbsp; **Description** - The Data Science lead in a project administration role for the AML workspace. 
<br/>&nbsp;&nbsp; **Type** - Person.<br/>
&nbsp;&nbsp; **Project Specific** - Yes. 
<br/>&nbsp;&nbsp; **Notes** - This role would also have break/fix responsibility for the ML models and packages used.<br/>

##### 6 - Project Owner (Bus Owner)
&nbsp;&nbsp; **Description** - The Business stakeholders responsible for the AML workspace based upon data ownership. 
<br/>&nbsp;&nbsp; **Type** - Person.<br/>
&nbsp;&nbsp; **Project Specific** - Yes. 
<br/>&nbsp;&nbsp; **Notes** - This role is read-only for the AML workspace configuration and components in development. Production coverage will be provided by the data governance application.<br/>

##### 7 - Platform Technical Support
&nbsp;&nbsp; **Description** - The Technical support staff responsible for break/fix activities across the platform. This role would cover infrastructure, service, etc. But not the ML models, packages or data. These elements remain under the [Data Scientist/ML Engineer](secure_aml_security.md#1---data-scientistml-engineer) role's responsibility. 
<br/>&nbsp;&nbsp; **Type** - Person.<br/>
&nbsp;&nbsp; **Project Specific** - No. 
<br/>&nbsp;&nbsp; **Notes** - While the role group is permanent, membership is only transient, based upon a Privileged Identity Management ([PIM](https://learn.microsoft.com/entra/id-governance/privileged-identity-management/pim-configure)) process for time boxed, evaluated access.<br/>

##### 8 - Model End User
&nbsp;&nbsp; **Description** - The End consumers of the ML Model. This role could be a downstream process or an individual. 
<br/>&nbsp;&nbsp; **Type** - Person and Process.<br/>
&nbsp;&nbsp; **Project Specific** - Yes. 

##### 9 - CI/CD processes
&nbsp;&nbsp; **Description** - The compute processes that releases/rolls back change across the platform environments.
<br/>&nbsp;&nbsp; **Type** - Process.<br/>
&nbsp;&nbsp; **Project Specific** - No. 

##### 10 - AML Workspace
&nbsp;&nbsp; **Description** - The [managed identities](/azure/machine-learning/how-to-setup-authentication?view=azureml-api-2&tabs=sdk) used by an AML workspace to interact with other parts of Azure.
<br/>&nbsp;&nbsp; **Type** - Person.<br/>
&nbsp;&nbsp; **Project Specific** - No. 
<br/>&nbsp;&nbsp; **Notes** - This persona represents the various services that make up an AML implementation, which interact with other parts of the platform, such as, the development workspace connecting with the development data store, etc.<br/>

##### 11 - Monitoring Processes
&nbsp;&nbsp; **Description** - The compute processes which monitor & alert based upon platform activities. 
<br/>&nbsp;&nbsp; **Type** - Process.<br/>
&nbsp;&nbsp; **Project Specific** - No. 

##### 12 - Data Governance Processes
&nbsp;&nbsp; **Description** - The compute process that scans the ML project and datastores for data governance.
<br/>&nbsp;&nbsp; **Type** - Process.<br/>
&nbsp;&nbsp; **Project Specific** - No. 
<br/>

#### Identity RBAC – Control Plane

The [Control plane](/azure/azure-resource-manager/management/control-plane-and-data-plane#control-plane) is used to manage the resource level objects with a subscription.

The Persona based identity RBAC design for the control plane for each environment can be described as;
##### Production

:::image type="content" source="_images/secureAML_ControlPrd.png" lightbox="_images/secureAML_ControlPrd.png" alt-text="Diagram showing the secure AML Identity RBAC – Control Plane - Production." border="false":::

##### Development

:::image type="content" source="_images/secureAML_ControlPre.png" lightbox="_images/secureAML_ControlPre.png" alt-text="Diagram showing the secure AML Identity RBAC – Control Plane - Development." border="false":::

Cell Colour = Access Period Granted. Green = Life of Project, Orange = Temporary, Just-in-time.

**Key**
<br/>
**[Standard Roles](/azure/role-based-access-control/built-in-roles#general)**
<br/>&nbsp;&nbsp; R = Reader.<br/>
&nbsp;&nbsp; C = Contributor.
<br/>&nbsp;&nbsp; O = Owner.<br/>
**[Component Specific Roles](/azure/role-based-access-control/built-in-roles/ai-machine-learning)**
<br/>&nbsp;&nbsp; ADS = Azure Machine Learning Data Scientist.<br/>
&nbsp;&nbsp; ACO = Azure Machine Learning Compute Operator.
<br/>&nbsp;&nbsp; ARU = Azure Machine Learning Registry User.<br/>
&nbsp;&nbsp; ACRPush = Azure Container Registry Push.
<br/>&nbsp;&nbsp; DOPA = DevOps Project Administrators.<br/>
&nbsp;&nbsp; DOPCA = DevOps Project Collection Administrators.
<br/>&nbsp;&nbsp; LAR = Log Analytics Reader. <br/>
&nbsp;&nbsp; LAC = Log Analytics Contributor.
<br/>&nbsp;&nbsp; MR = Monitoring Reader. <br/>
&nbsp;&nbsp; MC = Monitoring Contributor.

> [!IMPORTANT]
> Once a model has been productionized using one or more [Azure AI Services](/azure/ai-services/) API’s, service specific [built-in roles](/azure/role-based-access-control/built-in-roles) should be implemented into that project.
<br/>

#### Identity RBAC – Data/Model Plane

The [Data plane](/azure/azure-resource-manager/management/control-plane-and-data-plane#data-plane) is used to manage the capabilities exposed by a resource.

The Persona based identity RBAC design for the data plane for each environment can be described as;

##### Production

:::image type="content" source="_images/secureAML_DataPrd.png" lightbox="_images/secureAML_DataPrd.png" alt-text="Diagram showing the secure AML Identity RBAC – Data Plane - Production." border="false":::

##### Development

:::image type="content" source="_images/secureAML_DataPre.png" lightbox="_images/secureAML_DataPre.png" alt-text="Diagram showing the secure AML Identity RBAC – Data Plane - Development." border="false":::

Cell Colour = Access Period Granted. Green = Life of Project, Orange = Temporary, Just-in-time.

**Key**
<br/>
**[Standard Roles](/azure/role-based-access-control/built-in-roles#general)**
<br/>&nbsp;&nbsp; R = Reader.<br/>
&nbsp;&nbsp; C = Contributor.
<br/>&nbsp;&nbsp; O = Owner.<br/>
**[Component Specific Roles](/azure/role-based-access-control/built-in-roles/ai-machine-learning)**
<br/>&nbsp;&nbsp; ADS = Azure Machine Learning Data Scientist.<br/>
&nbsp;&nbsp; ACO = Azure Machine Learning Compute Operator.
<br/>&nbsp;&nbsp; ARU = Azure Machine Learning Registry User.<br/>
&nbsp;&nbsp; ACRPush = Azure Container Registry Push.
<br/>&nbsp;&nbsp; DOPA = DevOps Project Administrators.<br/>
&nbsp;&nbsp; DOPCA = DevOps Project Collection Administrators.
<br/>&nbsp;&nbsp; KVA = Key Vault Administrator. <br/>
&nbsp;&nbsp; KVA = Key Vault Administrator.

> [!IMPORTANT] 
> - Data plane controls are additive to the Control plane, i.e. they build on top of them.
> - The Data plane controls vary depending on the specific AI Service selected, its recommended to take to most restrictive scope matched with the most appropriate built-in role available for the role/task requirements.

## Contributors

*This article is maintained by Microsoft. It was originally written by the following contributors.*

Principal authors:

- [Scott Donohoo](https://www.linkedin.com/in/scottdonohoo) | Senior Cloud Solution Architect
- [Moritz Steller](https://www.linkedin.com/in/moritz-steller-426430116/) | Senior Cloud Solution Architect

Other contributors:

- [Scott Mckinnon](https://www.linkedin.com/in/scott-mckinnon-96756a83) | Cloud Solution Architect
- [Nicholas Moore](https://www.linkedin.com/in/nicholas-moore/) | Cloud Solution Architect
- [Darren Turchiarelli](https://www.linkedin.com/in/darren-turchiarelli/) | Cloud Solution Architect
- [Leo Kozhushnik](https://www.linkedin.com/in/leo-kozhushnik-ab16707/) | Cloud Solution Architect

*To see non-public LinkedIn profiles, sign in to LinkedIn.*

## Next steps

- [What is Azure Pipelines?](/azure/devops/pipelines/get-started/what-is-azure-pipelines)
- [Azure Arc overview](/azure/azure-arc/overview)
- [What is Azure Machine Learning?](/azure/machine-learning/overview-what-is-azure-machine-learning)
- [Data in Azure Machine Learning](/azure/machine-learning/concept-data)
- [Azure MLOps (v2) solution accelerator](https://github.com/Azure/mlops-v2)
- [End-to-end machine learning operations (MLOps) with Azure Machine Learning](/training/paths/build-first-machine-operations-workflow)
- [Introduction to Azure Data Lake Storage Gen2](/azure/storage/blobs/data-lake-storage-introduction)
- [Azure DevOps documentation](/azure/devops)
- [GitHub Docs](https://docs.github.com)
- [Azure Synapse Analytics documentation](/azure/synapse-analytics)
- [Azure Event Hubs documentation](/azure/event-hubs)

## Related resources

- [Choose a Microsoft cognitive services technology](../../data-guide/technology-choices/cognitive-services.md)
- [Natural language processing technology](../../data-guide/technology-choices/natural-language-processing.yml)
- [Compare the machine learning products and technologies from Microsoft](../../ai-ml/guide/data-science-and-machine-learning.md)
- [How Azure Machine Learning works: resources and assets (v2)](/azure/machine-learning/concept-azure-machine-learning-v2)
- [What are Azure Machine Learning pipelines?](/azure/machine-learning/concept-ml-pipelines)
- [Machine learning operations (MLOps) framework to upscale machine learning lifecycle with Azure Machine Learning](mlops-technical-paper.yml)
- [What is the Team Data Science Process?](/azure/architecture/data-science-process/overview)
