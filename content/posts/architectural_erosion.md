+++
title = 'Architectural Erosion'
date = 2026-10-29T18:36:30+08:00
draft = true
+++

I wrote this article to summarize what I learned from the past year's work. I use the word ‘erosion’ because architectures often start with a clean design, but as development continues, they gradually drift from their original intent and become a new source of pain for the team.

## Monorepos are not as beautiful as they are described

In my experience, working with a company-wide monorepo can be very painful. Monorepos are often described as a way for everyone in a company to easily share code and use unified tooling.
But in practice:

1. As the team grows, one small change may affect all applications, and the entire CI pipeline may take hours to run. This slows down feedback and adds unnecessary costs.
2. In our monorepo, shared libraries are not versioned independently. For example, if you change a library that another team also uses, you may also need to update that team's codebase and involve them in the review. Tracking down reviewers and waiting for approvals can turn a small change into a lengthy process.
3. Companies often have outdated applications that no one actively maintains but that still serve real users. Changing the underlying tooling can be a nightmare. For example, our monorepo uses webpack and Jest, but Jest does not support the ESM setup we need, so we have to switch to Vitest. Some applications now use Vitest, while others still use Jest. Without someone responsible for completing the migration, unified tooling can remain just a beautiful dream.

## Microservices organization

I consider microservices one of the best architectural ideas of the last ten years. They help decouple systems and let teams develop independently, but in a previous role, I also saw problems emerge when we developed microservices without clear boundaries.

As the team grows, we keep introducing applications with different responsibilities. If we do not define data ownership clearly, even a foundational service can become dependent on higher-level business services. For example, imagine an online marketplace where one account can have both a customer profile and a seller profile. The account service does not store the links between an account and those profiles. Instead, whenever it needs to identify which profiles an account has, it queries both the customer service and the seller service using the account ID. A basic account lookup now depends on two business services being available, even when the caller only needs the profile IDs. This makes the account service harder to reuse and allows failures in business services to affect a foundational capability.

My proposed solution is to define clear service layers and place ownership of shared relationships in the appropriate foundational service. In this example, the account service should own the links between an account and its customer and seller profiles, so it can return those links directly without querying the business services. The customer and seller services still own their full profile records. If a screen needs those details, the backend for frontend (BFF) can use the returned IDs to query the relevant services. When shared account information, such as an email address or preferred name, changes, the account service publishes an event so the business services can update their local copies. Across the organization, synchronous requests should flow only from higher layers to lower layers, with no calls to services in the same layer; lower layers publish events to communicate changes upward. The key is to align data ownership with that dependency direction. This removes the business-service calls from the basic account lookup, while keeping requests for full business details explicit and limited to the flows that need them. Event-driven updates also mean that local copies may briefly lag behind the account service.
