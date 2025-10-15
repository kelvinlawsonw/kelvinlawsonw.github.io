+++
title = 'Azure Look-Alikes Explained: Quick Choices for Real-World Designs'
date = 2025-10-14T21:56:26-05:00
draft = false
tags = ["azure", "cloud", "devops", "networking" ]
+++

If you’ve ever paused mid design wondering __“which Azure thing is this again?”__ you’re not alone. The ecosystem is huge, the names are similar, and even senior engineers double-check. This guide clears it up with quick, practical comparisons—so you can choose faster and ship with confidence.

### Service Endpoints vs Private Endpoints
+ Service Endpoint: Your subnet can reach a PaaS service at its public IP, but over Microsoft’s backbone. You “allow” that subnet on the service side.
+ Private Endpoint: You get a private IP for one specific resource/sub-resource (e.g., Storage Blob vs Queue), injected into your VNet; traffic never hits the public IP


###  Managed Identity vs Service Principal
+ Managed Identity (MI): Azure creates/rotates credentials for you; identity is tied to a particular resource (system-assigned) or stands alone and reusable (user-assigned).
+ Service Principal (SP): Identity for an app registration you create; you manage the credentials (secret or cert) or use workload identity federation (no long-lived secret) for CI/CD processes. 

### Azure Network Security Groups(NSG) vs Azure Firewall
+ NSG: Stateful L3/L4 allow/deny rules on subnets with lightweight isolation. These are typically per subnet and offer simple segmentation.
+ Azure Firewall: Managed, centralized L3–L7 firewall with DNAT/SNAT, FQDN/URL filtering, and Premium features like TLS inspection. It's Hub-and-spoke friendly, offers Egress governance, application-aware rules and centralized NAT.

### Availability Sets vs Availability Zones
+ Availability Set: Spreads VMs across fault and update domains inside one datacenter (up to 3 FDs / 20 UDs). Low latency between members; protects from rack/host failures and co-ordinated updates. It's ideal for single-site, latency-sensitive tiers (e.g., stateful clusters).
+ Availability Zones: Place resources across separate datacenters in a region. Two+ zone-spanned VMs get a 99.99% VM connectivity SLA. It's important to mention that data transfer between zones in the same region is now free. It's ideal for higher fault isolation and regional resilience.

