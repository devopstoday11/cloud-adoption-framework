---
title: Transitioning existing Azure environments to enterprise-scale 
description: Onboard existing environments to an enterprise-scale architecture
author: BrianBlanchard
ms.author: brblanch
ms.date: 10/15/2020
ms.topic: conceptual
ms.service: cloud-adoption-framework
ms.subservice: ready
ms.custom: csu
---

# Transition existing Azure environments to Enterprise-Scale

We recognize that most organizations may have an existing footprint in Azure, one or more subscriptions, and potentially an existing structure of their management groups. Depending on their initial business requirements and scenarios, Azure resources such as hybrid connectivity (for example with Site-to-Site VPN and/or ExpressRoute) may have been deployed.  

This article helps organizations to navigate the right path based on an existing Azure environment transitioning into Enterprise-Scale. This article also describes considerations for moving resources in Azure (for example, moving a subscription from one existing management group to another management group), which will help you evaluate and plan for transitioning your existing Azure environment to Enterprise-Scale landing zones.

## Moving resources in Azure

Some resources in Azure can be moved post creation, and there are different approaches organizations can take subject to users RBAC permissions at – and across scopes. The following table outlines which resources can be moved, at which scope, and the pros/cons associated with each.

| Scope | Destination | Pros | Cons |
|--|--|--|--|
| Resources in resource groups | Can be moved to new resource group in same or different subscription  | Allows you to modify resource composition in a resource group after deployment | - Not supported by all resourceTypes <br> - Some resourceTypes have specific limitations or requirements <br> - ResourceIds are updated and impacts existing monitoring, alerts, and control plane operations <br> - Resource groups are locked during the move period <br> - Requires assessment of policies and RBAC pre and post-move operation |
| Subscriptions in a tenant  | Can be moved to different management groups, and different tenants | No impact to existing resources within the subscription, as no resourceId’s will be changed | Requires assessment of policies and RBAC pre and post-move operation |

To understand which move strategy you should use, we will go through examples of both:

## Subscription move

The common use cases for moving subscriptions are to organize subscriptions into management groups or when transfering subscriptions to a new Azure Active Directory tenant. Subscription moves for enterprise-scale focuses on moving subscriptions to management groups. Moving a subscription to a new tenant is mainly for [transferring billing ownership](https://docs.microsoft.com/azure/cost-management-billing/manage/billing-subscription-transfer). 

### RBAC requirements

To assess a subscription prior to a move, it is important that the user has the appropriate RBAC such as being an Owner on the subscription (direct roleAssignment), and has write permission on the target management group (built-in roles that support this is Owner, Contributor, Management Group Contributor).

If the user has an inherited Owner permission on the subscription from an existing management group, the subscription can only be moved to the management group where the user has been assigned the Owner role.

### Policy

Existing subscriptions may be subject to Azure policies assigned either directly, or at the management group where they are currently located. It is important to assess current policies, and the policies that may exist in the new management group/management group hierarchy.

Azure Resource Graph can be used to perform an inventory of existing resources and compare their configuration with the policies existing at the destination.

Once subscriptions are moved to a management group with existing RBAC and policies in place, consider the following options:

- Any RBAC that is inherited to the moved subscriptions can take up to 30 minutes before the user tokens in the management group cache are refreshed. To expedite this process, you can refresh the token by signing out and in or request a new token.
- Any policy where the assignment scope includes the moved subscriptions, will perform audit operations only on the existing resources. More specifically:
  - Any existing resource in the subscription subject to **deployIfNotExists** policy effect will appear as non-compliant and will not be remediated automatically but requires user interaction to perform the remediation manually.
  - Any existing resource in the subscription subject to **deny** policy effect will appear as non-compliant and will not be rejected. User must manually mitigate this result as appropriate.
  - Any existing resource in the subscription subject to **append** and **modify** policy effect will appear as non-compliant and requires user interaction to mitigate.
  - Any existing resource in the subscription subject to **audit** and **auditIfNotExist** will appear as non-compliant and requires user interaction to mitigate.
- All new writes to resources in the moved subscription will be subject to the assigned policies at real-time as normal.

## Resource move

The primary use cases to perform a resource move is when you want to consolidate resources into the same resource group if they share the same life-cycle, or move resources to a different subscription due to cost, ownership, or RBAC requirements.

When performing a resource move, both the source resource group and the target resource group are locked (this lock will not affect any of the resources in the resource group) during the move operation, meaning you cannot add, update, or delete resources in the resource groups. A resource move operation will not change the location of the resources.

### Before you move resources

Prior to a move operation, you must verify that the [resources in scope are supported](https://docs.microsoft.com/azure/azure-resource-manager/management/move-support-resources) as well as assessing their requirements and dependencies. For instance, moving a peered virtual network requires you to disable virtual network peering first, and re-enable the peering once the move operation has completed. This disable/re-enable dependency requires planning upfront to understand the impact to any existing workload that may be connected to your virtual networks.

### Post-move operation

When the resources are moved into a new resource group in the same subscription, any inherited RBAC and policies from management group or/and subscription scope will still apply. If you move to a resource group in a new subscription – where the subscription may be subject to other RBAC and policy assignment, same guidance applies as to the move subscription scenario to validate the resource compliance and access controls.
