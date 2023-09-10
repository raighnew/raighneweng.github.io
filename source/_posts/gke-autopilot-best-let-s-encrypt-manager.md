---
title: Best Let's Encrypt Manager for Kong + GKE Autopilot 
date: 2023-09-10 09:37:37
categories: Backend
tags: [GKE, Kubernetes, Kong]
---

In this article, I will not delve into the basics of TLS/SSL and how SSL works. Additionally, if you are using GKE default ingress, you also may not encounter this issue. Instead, I will focus on why I chose to use Let's Encrypt and how to set up automatic renewal for it in Google Kubernetes Engine (GKE) Autopilot and third-party ingress using Kong.

<!--more-->

## Let's Encrypt is free but also safety

Before 2014, when Let's Encrypt was established, if you wanted to apply an SSL certificate, you needed to follow these steps:

1. Configure a DNS TXT record following the third-party guidelines.
2. Apply for and download the SSL certificate from a third-party portal.
3. Upload the SSL certificate to your cloud instance and configure Nginx.
4. Set up an alert to remind yourself to renew the certificate after 2 years.

Among these steps, the most challenging one is Step 4. It can be exceptionally difficult to remember to renew something two years in the future, especially within a large company. In such environments, personnel changes are common, and if the responsible individual resigns or overlooks the renewal during a handover, it can create a significant issue.

However, Let's Encrypt issues certificates with a validity period of only 90 days. Despite this seemingly short timeframe, this approach brings several notable benefits:

1. Change Private Key Every 90 Days for Enhanced Safety.
2. This deliberate short duration compels users to implement an automated renewal mechanism, effectively reducing the potential for human errors. 

Let's Encrypt is a great company.

## GKE Autopilot

[Cert-Manager](https://cert-manager.io/docs/) is a popular certificate manager solution for Kubernetes, and you can find recommendations for it in many related projects. It also supports various issuers. However, on GKE Autopilot, it may not be the optimal choice.

GKE Autopilot charges us based on CPU and memory usage, with each pod requiring a minimum of 0.25 vCPU and 0.5 GiB of memory. In this context, using cert-manager is not feasible, as it consists of three pods that collectively demand 0.75 vCPU and 1.5 GiB of memory. These resource requirements are beyond what can be accommodated within GKE Autopilot's constraints.

So, my specific requirements for this scenario are as follows:

1. No CRDs(Custom Resource Definitions) to operate
2. Single service/pod in my cluster

Considering these requirements, I discovered [Kcert](https://github.com/nabsul/kcert) to be the optimal solution for my needs. 

I have created two pull requests (PRs) to add support for custom resource limits and to enable the use of multiple kcert instances for different ingress classes.

Here is an example configuration for an Ingress resource:
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: test1-ingress
  labels:
    kcert.dev/ingress: "managed"
spec:
  tls:
  - hosts:
    - test1.kcert.dev
    secretName: test1-tls
  rules:
  - host: test1.kcert.dev
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: hello-world
            port:
              number: 80
```

With kcert, it monitors newly created Ingress resources that match the `kcert.dev/ingress: "managed"` label. When it detects a new Ingress record, kcert automatically initiates an ACME challenge Ingress to fulfill the ACME challenge. After a few minutes, it saves the TLS certificate to a secret defined by the `secretName`.

It is very simple to manage and lightweight.

Reference Links:
[Resource requests in Autopilot](https://cloud.google.com/kubernetes-engine/docs/concepts/autopilot-resource-requests)
[Kcert](https://github.com/nabsul/kcert)
[Cert-Manager](https://cert-manager.io/docs/)
[Everything about TLS/SSL](https://www.kawabangga.com/posts/5330)
