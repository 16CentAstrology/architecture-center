---
title: Architectural approaches for identity in multitenant solutions
titleSuffix: Azure Architecture Center
description: This article describes the approaches for managing identities in a multitenant solution.
author: johndowns
ms.author: jodowns
ms.date: 05/08/2022
ms.topic: conceptual
ms.service: architecture-center
ms.subservice: azure-guide
products:
  - azure
  - azure-active-directory
  - azure-active-directory-b2c
categories:
  - identity
ms.category:
  - fcp
ms.custom:
  - guide
---

# Architectural approaches for identity in multitenant solutions

Almost all multitenant solutions require an identity system. In this article, we discuss common components of identity, including both authentication and authorization, and discuss how these can be applied in a multitenant solution.

> [!NOTE]
> Review [Architectural considerations for identity in a multitenant solution](../considerations/identity.md) to learn more about the key requirements and decisions you need to make before you start to build an identity system for a multitenant solution.

## Authentication

Authentication is the process by which a user's identity is established. When you build a multitenant solution, there are special considerations and approaches for several aspects of the authentication process.

### Federation

You might need to federate with other identity providers. Federation can be used to enable several scenarios, including:

- Social login, such as by enabling users to use a Google or personal Microsoft account.
- Tenant-specific directories, such as by enabling tenants to federate your application with their own identity providers so they don't need to manage accounts in multiple places.

For general information about federation, see the [Federated Identity pattern](../../../patterns/federated-identity.yml).

If you choose to support tenant-specific identity providers, ensure you clarify which services and protocols you need to support. For example, will you support the OpenID Connect protocol as well as the Security Assertion Markup Language (SAML) protocol? Or, will you only support federating with Azure Active Directory instances?

When you build your own identity provider, consider any scale and limits that might apply. For example, if you use Azure Active Directory (Azure AD) B2C as your own identity provider, you might need to deploy custom policies to federate with certain types of tenant identity providers. Azure AD B2C [limits the number of custom policies](/azure/active-directory-b2c/service-limits?pivots=b2c-custom-policy#azure-ad-b2c-configuration-limits) that you can deploy, which might limit the number of tenant-specific identity providers that you can federate with.

You can also consider providing federation as a feature that only applies to customers at a higher [product tier](../considerations/pricing-models.md#feature--and-service-level-based-pricing).

### Single sign-on

Single sign-on experiences enable users to move between applications seamlessly, without being prompted to reauthenticate at each point.

When a user visits an application, the application directs them to an identity provider, which has access to their existing login session and issues a new token for the current application without requiring the user to interact with the login process. A federated identity model support single sign-on experiences by enabling users to use a single identity across multiple applications.

In a multitenant solution, you might also enable another form of single sign on. If a user is authorized to work with data for multiple tenants, you might need to provide a seamless experience when the user changes their context from one tenant to another. Consider whether you need to support seamless transitions between tenants, and if so, whether your identity provider needs to reissue tokens with specific tenant claims.

### Sign-in risk evaluation

Modern identity platforms support risk evaluation during the sign-in process. For example, if a user signs in from an unusual location or device, the authentication system might require additional identity checks, such as MFA, before allowing the sign-in request to proceed.

Consider whether your tenants might have different risk policies that need to be applied during the authentication process. For example, if you have some tenants in a highly regulated industry, they might have different risk profiles and requirements to tenants who work in less regulated environments. Or, you might choose to allow tenants at higher pricing tiers to specify more restrictive sign-in policies than tenants who purchase a lower tier of your service.

If you need to support different risk policies for each tenant, your authentication system needs to know which tenant the user is signing into so that the correct policies can be applied.

Alternatively, if you federate to tenants' own identity providers, then their own risky sign-in mitigation policies can be applied, and they can control the policies and controls that should be enforced.

### Impersonation

Impersonation enables a user to assume the identity of another user without using that user's credentials.

In general, impersonation is dangerous, and it can be difficult to implement and control. However, in some scenarios, impersonation is a requirement. For example, if you operate software as a service (SaaS), your helpdesk personnel might need to assume a user's identity so that they can sign in as the user and troubleshoot an issue.

Some identity platforms support impersonation, either as a built-in feature or through the use of custom code. For example, [Azure AD B2C provides a sample implementaton of an impersonation flow](https://github.com/azure-ad-b2c/samples/tree/master/policies/impersonation).

### Token enrichment and authentication flow customization

In a multitenant solution, you might need to integrate your authentication processes with tenants' systems. Integration can happen for several purposes, including injecting custom claims into a login token (*token enrichment*), or triggering events in a tenant's own systems.

Whenever you integrate with a tenant's system, it's a good practice to build a common integration approach instead of trying to build customized integration approaches for each tenant. For example, if your tenants need to enrich token claims, you might define an API that each tenant must implement, with a specific payload format. You can then agree to send a request to that tenant's API during each sign-in, and the tenant agrees to send a response with another specified payload format, which you then turn into a set of claims to add to the token.

## Authorization

Authorization is the process of determining what a user is allowed to do.

Authorization can be implemented in several places, including:

- **In your identity provider.** For example, if you use Azure AD as your identity provider, features like [app roles](/azure/active-directory/develop/howto-add-app-roles-in-azure-ad-apps) and [groups](/azure/active-directory/fundamentals/active-directory-groups-create-azure-portal) can be used to enforce authorization rules.
- **In your application.** You can build your own authorization logic, and store information about what each user can do in a database or similar storage system.

In a multitenant solution, roles are generally assigned and managed by the tenant or customer, not by you as the vendor of the multitenant system.

For more information, see [Application roles](../../../multitenant-identity/app-roles.md).

### Add tenant identity and role information to tokens

Consider which part, or parts, of your solution should perform authorization requests, including determining whether a user is allowed to work with data from a specific tenant.

A common approach is for your identity system to embed a tenant identifier claim into a token. This enables your application to inspect the claim and verify that the user is working with the tenant that they are allowed to access. If you use the role-based security model, then you might choose to extend the token with information about the role a user has within the tenant.

However, if a single user is allowed to access multiple tenants, you might need to provide a way for your users to signal which tenant they plan to work with during the login process so that the token can include the correct tenant identifier claim and role for that tenant. You also need to consider how users can switch between tenants, which requires issuing a new token.

#### Application-based authorization

An alternative approach is to make the identity system agnostic to tenant identifiers and roles. Each user is identified using their credentials or a federation relationship, and tokens don't include a tenant identifier claim. A separate list or database contains which users have been granted access to each tenant. Then, the application tier can verify whether the specified user should be allowed to access the data for a specific tenant based on looking up that list.

## Use Azure AD or Azure AD B2C

Microsoft provides Azure Active Directory (Azure AD) and Azure AD B2C, which are managed identity platforms that you can use within your own multitenant solution.

Many multitenant solutions are software as a service (SaaS). Your choice of whether to use Azure AD or Azure AD B2C depends in part on how you define your tenants or customer base.

- If your tenants or customers are organizations, you might consider using Azure AD. You can then create a [multitenant application](/azure/active-directory/develop/single-and-multi-tenant-apps) to make your solution available to other Azure AD directories. You can even list your solution in the [Azure Marketplace](https://azuremarketplace.microsoft.com/marketplace/apps/category/azure-active-directory-apps) and make it easily accessible to organizations who use Azure AD.
- If your tenants or customers don't use Azure AD, or if they are individuals rather than organizations, then consider using Azure AD B2C. Azure AD B2C provides a set of features to control how users sign up and sign in. For example, you can restrict access to your solution just to users that you have already invited, or you might allow for self-service sign-up. Use [custom policies](/azure/active-directory-b2c/active-directory-b2c-overview-custom) in Azure AD B2C to fully control how users interact with the identity platform. You can use [custom branding](/azure/active-directory-b2c/customize-ui-overview), and you can [federate Azure AD B2C with your own Azure AD tenant](/azure/active-directory-b2c/active-directory-b2c-setup-oidc-azure-active-directory) to enable your own staff to sign in. Azure AD B2C also enables [federation with other identity providers](/azure/active-directory-b2c/tutorial-add-identity-providers).
- Some multitenant solutions are intended for both situations listed above - some tenants might have their own Azure AD tenants, and others might not. You can also use Azure AD B2C for this scenario, and use [custom policies to allow user sign-in from a tenant's Azure AD directory](/azure/active-directory-b2c/active-directory-b2c-setup-commonaad-custom).

## Antipatterns to avoid

### Building or running your own identity system

Building a modern identity platform is complex. There are a range of protocols and standards to support, and it's easy to incorrect implement a protocol and expose a security vulnerability. Standards and protocols change, and you also need to continually update your identity system to mitigate attacks and support recent security features. It's also important to ensure that an identity system is resilient, because any downtime can have severe consequences for the rest of your solutoin. Additionally, in most situations, identity doesn't add benefit to the business and is simply a necessary part of implementing a multitenant service.

When you run your own identity system, you need to store passwords and other credentials. By storing credentials, you create a tempting target for attackers. Even hashing and salting passwords is often insufficient protection, because the computational power available to attackers can make it possible to compromise these forms of credentials.

Running a modern identity system also means you are responsible for generating and distributing MFA or one-time password (OTP) codes, which in turn requires that you have a mechanism to distribute these codes by using SMS or email.

Instead of building or running your own identity system, it's a good practice to use an off-the-shelf service or component. For example, consider using Azure Active Directory (Azure AD) or Azure AD B2C, which are managed identity platforms. Managed identity platform vendors take responsibility to operate the infrastructure for their platforms, and typically support the current identity and authentication standards.

### Failing to consider tenants' requirements

Tenants often have strong opinions about how identity should be managed for the solutions they use. For example, many enterprise customers require federation with their own identity providers to enable single sign-on experiences and to avoid managing multiple sets of credentials. Other tenants might require multifactor authentication or other forms of protection around the sign-in processes. If you haven't designed for these requirements, it can be challenging to retrofit them later.

Ensure you understand your tenants' identity requirements before you finalize the design of your identity system. Review [Architectural considerations for identity in a multitenant solution](../considerations/identity.md) to understand some specific requirements that often emerge.

### Conflating users and tenants

It's important to clearly consider how your solution defines a user and a tenant. In many situations, the relationship can be complex. For example, a tenant might contain multiple users, and a single user might join multiple tenants.

Ensure you have a clear process for tracking the tenant context within your application and requests. In some situations, this process might require that you include a tenant identifier in every access token, and that you validate the tenant identifier on each request. In other situations, you might need to store the tenant authorization information separately from the user identities and use a more complex authorization system to manage which users can perform which operations against which tenants.

### Conflating role and resource authorization

It's important that you select an appropriate authorization model for your solution. Role-based security approaches can be simple to implement, but resource-based authorization provides more fine-grained control. Consider your tenants' requirements, and whether your tenants need to authorize some users to access specific parts of your solution and not others.

### Failing to write audit logs

Audit logs are an important tool for understanding your environment and how users are using your system. By auditing every identity-related event, you can often determine whether your identity system is under attack, and you can review how your system is being used. Ensure you write and store audit logs within your identity system. Consider whether your solution's identity audit logs should be made available to tenants to review.

## Next steps

Review [Architectural considerations for identity in a multitenant solution](../considerations/identity.md).
