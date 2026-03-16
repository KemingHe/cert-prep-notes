# Study Notes - EC2 Scalability, ELB, and ASG

> **Last Updated**: 2026-03-16 by Keming He

## Table of Contents

- [Study Notes - EC2 Scalability, ELB, and ASG](#study-notes---ec2-scalability-elb-and-asg)
  - [Table of Contents](#table-of-contents)
  - [Scalability Concepts](#scalability-concepts)
    - [Scalability](#scalability)
    - [Elasticity](#elasticity)
    - [Agility](#agility)
    - [High Availability](#high-availability)
  - [Elastic Load Balancing Overview](#elastic-load-balancing-overview)
  - [ELB Types Comparison](#elb-types-comparison)
  - [Application Load Balancer (ALB)](#application-load-balancer-alb)
  - [Network Load Balancer (NLB)](#network-load-balancer-nlb)
  - [Gateway Load Balancer (GWLB)](#gateway-load-balancer-gwlb)
  - [Auto Scaling Groups Overview](#auto-scaling-groups-overview)
  - [Launch Templates vs Launch Configurations](#launch-templates-vs-launch-configurations)
  - [ASG Scaling Strategies](#asg-scaling-strategies)
    - [Manual Scaling](#manual-scaling)
    - [Dynamic Scaling](#dynamic-scaling)
    - [Scheduled Scaling](#scheduled-scaling)
    - [Predictive Scaling (ML-Based)](#predictive-scaling-ml-based)
  - [ASG and ELB Integration](#asg-and-elb-integration)
    - [Health Check Types](#health-check-types)
    - [Health Check Grace Period](#health-check-grace-period)
    - [Instance Warmup](#instance-warmup)
  - [Multi-AZ Deployment](#multi-az-deployment)
  - [Shared Responsibility Model](#shared-responsibility-model)
    - [AWS Responsibilities](#aws-responsibilities)
    - [Customer Responsibilities](#customer-responsibilities)
  - [Best Practices](#best-practices)
  - [References](#references)

## Scalability Concepts

Understanding AWS scalability terminology is essential for the Cloud Practitioner exam - these terms appear frequently and have specific AWS definitions.

### Scalability

The ability of a system to adapt and meet increasing demand over time as it grows.

| Type | Direction | Description | Example |
| :--- | :--- | :--- | :--- |
| **Vertical** (scale up/down) | Single resource | Increase or decrease individual resource capacity | t3.medium to t3.xlarge |
| **Horizontal** (scale out/in) | Multiple resources | Add or remove resources to distribute load | 2 instances to 10 instances |

AWS recommends [horizontal scaling to reduce single points of failure](https://docs.aws.amazon.com/wellarchitected/latest/reliability-pillar/design-principles.html) - replacing one large resource with multiple smaller ones.

### Elasticity

The ability to [acquire resources as you need them _and release resources when you no longer need them_](https://docs.aws.amazon.com/whitepapers/latest/cost-optimization-automating-elasticity/introduction.html) - automatically.

| Concept | Key Differentiator |
| :--- | :--- |
| Scalability | Capacity to grow (may be manual, often one direction) |
| Elasticity | Automatic + bidirectional (scale out AND scale in) |

Services _with built-in elasticity_: S3, SQS, SNS, Aurora Serverless, Athena

Services _requiring configuration_: EC2/ECS via Auto Scaling, [DynamoDB in provisioned capacity mode](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/HowItWorks.ReadWriteCapacityMode.html) (on-demand mode provides built-in elasticity)

### Agility

In AWS context, agility means new IT resources are only a click away - reducing provisioning time from weeks to minutes. This enables low-cost experimentation and faster iteration. Listed as one of AWS's [Six Advantages of Cloud Computing](https://docs.aws.amazon.com/whitepapers/latest/aws-overview/six-advantages-of-cloud-computing.html).

### High Availability

Avoiding single points of failure by [deploying workloads across at least two Availability Zones](https://docs.aws.amazon.com/wellarchitected/latest/reliability-pillar/rel_fault_isolation_multiaz_region_system.html).

- Each AWS Region has 3+ AZs, geographically close but physically separated (independent power, cooling, networking)
- Inter-AZ latency is single-digit milliseconds (enables synchronous replication)
- Some services are multi-AZ by default (S3, DynamoDB, EFS); others require explicit configuration (RDS, ElastiCache)

> [↑ Back to Table of Contents](#table-of-contents)

---

## Elastic Load Balancing Overview

Load balancers are servers that forward internet traffic to multiple downstream targets (EC2 instances, containers, Lambda functions).

**Key benefits**:

- Spread load across multiple downstream targets
- Expose a single point of access (DNS) with SSL/TLS termination
- Perform health checks and seamlessly handle target failures
- Provide high availability across AZs

ELB is a managed service - AWS handles provisioning, scaling, and maintenance.

> [↑ Back to Table of Contents](#table-of-contents)

---

## ELB Types Comparison

| Type | Layer | Protocols | Static IP | Best For |
| :--- | :--- | :--- | :--- | :--- |
| **ALB** (Application) | 7 | HTTP, HTTPS (gRPC via HTTP/2, WebSocket via upgrade) | No | Web apps, microservices, content-based routing |
| **NLB** (Network) | 4 | TCP, UDP, TLS, QUIC | Yes (Elastic IP) | Ultra-high performance, static IPs, gaming |
| **GWLB** (Gateway) | 3 | GENEVE (all IP packets) | No | Third-party security appliances, firewalls |
| **CLB** (Classic) | 4 + 7 | HTTP, HTTPS, TCP, SSL | No | Legacy - migrate to ALB or NLB |

Classic Load Balancer still functions for VPC deployments but receives no new features - [migrate to ALB or NLB](https://docs.aws.amazon.com/elasticloadbalancing/latest/userguide/migrate-classic-load-balancer.html). EC2-Classic networking retirement was originally targeted for August 15, 2022, and was [confirmed complete on August 23, 2023](https://aws.amazon.com/blogs/aws/ec2-classic-is-retiring-heres-how-to-prepare/).

> [↑ Back to Table of Contents](#table-of-contents)

---

## Application Load Balancer (ALB)

Operates at Layer 7 (application layer) and is the best choice for HTTP/HTTPS workloads. See [Application Load Balancers user guide](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/application-load-balancers.html).

**Protocols**: HTTP, HTTPS, gRPC (via HTTP/2), WebSocket

**Key features**:

- Content-based routing (URL path, host header, HTTP headers, query strings)
- Native Lambda function targets
- User authentication (OIDC, Amazon Cognito)
- Weighted target groups for blue/green and canary deployments
- Request tracing via X-Amzn-Trace-Id header
- AWS WAF integration for web application firewall

**Cross-zone load balancing**: Always enabled at load balancer level (can disable at target group level)

**DNS**: Provides a static DNS name (URL) - no static IP

> [↑ Back to Table of Contents](#table-of-contents)

---

## Network Load Balancer (NLB)

Operates at Layer 4 (transport layer) for ultra-high performance and static IP requirements. See [Network Load Balancers user guide](https://docs.aws.amazon.com/elasticloadbalancing/latest/network/network-load-balancers.html).

**Protocols**: TCP, UDP, TLS, QUIC

**Performance characteristics**:

- Handles millions of requests per second
- [Ultra-low latency](https://aws.amazon.com/elasticloadbalancing/network-load-balancer/)
- [Preserves source IP](https://docs.aws.amazon.com/elasticloadbalancing/latest/network/edit-target-group-attributes.html) by default for instance-type targets; varies for IP-type targets (enabled for UDP/QUIC, disabled for TCP/TLS)

**Static IP support**:

- One static IP per Availability Zone
- Elastic IP addresses can be assigned (internet-facing)
- Private static IPs for internal NLBs
- [Subnets can be added/removed after creation](https://docs.aws.amazon.com/elasticloadbalancing/latest/network/availability-zones.html); Elastic IPs are assigned when enabling a subnet and cannot be changed without removing and re-adding the subnet

**Cross-zone load balancing**: Disabled by default (can enable)

**Routing**: Uses [flow hash algorithm](https://docs.aws.amazon.com/elasticloadbalancing/latest/userguide/how-elastic-load-balancing-works.html) based on protocol, source/destination IP and port; TCP includes sequence number (6-tuple), UDP uses 5-tuple

> [↑ Back to Table of Contents](#table-of-contents)

---

## Gateway Load Balancer (GWLB)

Operates at Layer 3 (network layer) for routing traffic through third-party security virtual appliances. See [Gateway Load Balancers user guide](https://docs.aws.amazon.com/elasticloadbalancing/latest/gateway/gateway-load-balancers.html).

**Protocol**: GENEVE (port 6081) - encapsulates all IP traffic

**Use cases**:

- Firewalls
- Intrusion Detection/Prevention Systems (IDS/IPS)
- Deep packet inspection
- Network traffic analysis

**How it works**: GWLB acts as a transparent network gateway. Traffic flows: source -> GWLB -> security appliance for inspection -> GWLB -> destination (if safe). GWLB does NOT terminate connections - it is pass-through only.

**Cross-zone load balancing**: Disabled by default

> [↑ Back to Table of Contents](#table-of-contents)

---

## Auto Scaling Groups Overview

[Auto Scaling Groups](https://docs.aws.amazon.com/autoscaling/ec2/userguide/auto-scaling-groups.html) automatically adjust EC2 instance count to match demand - scaling out when load increases and scaling in when load decreases.

**Core capacity settings**:

| Setting | Description |
| :--- | :--- |
| **Minimum size** | Floor - ASG never scales below this count |
| **Maximum size** | Ceiling - ASG never scales above this count |
| **Desired capacity** | Target count ASG maintains; adjusted by scaling policies |

**Key benefits**:

- Cost optimization by running at optimal capacity (no over-provisioning)
- Automatic ELB registration/deregistration during scaling events
- Automatic replacement of unhealthy instances
- Multi-AZ distribution for high availability

> [↑ Back to Table of Contents](#table-of-contents)

---

## Launch Templates vs Launch Configurations

| Feature | Launch Templates | Launch Configurations |
| :--- | :--- | :--- |
| **Status** | Current standard | Deprecated |
| **Versioning** | Yes - supports multiple versions | No - immutable after creation |
| **New instance types** | Fully supported | Not supported after Jan 1, 2023 |
| **Mixed instances** | Yes (Spot + On-Demand mixing) | No |

AWS recommends launch templates for all new deployments. Existing launch configurations should be [migrated to launch templates](https://docs.aws.amazon.com/autoscaling/ec2/userguide/migrate-to-launch-templates.html).

> [↑ Back to Table of Contents](#table-of-contents)

---

## ASG Scaling Strategies

### Manual Scaling

Manually adjust desired capacity via Console, CLI, or API. Use for testing or one-time adjustments.

### Dynamic Scaling

Automatically adjusts capacity based on CloudWatch metrics.

| Policy Type | Description | Use Case |
| :--- | :--- | :--- |
| **[Target tracking](https://docs.aws.amazon.com/autoscaling/ec2/userguide/as-scaling-target-tracking.html)** | Set target metric value; AWS manages alarms automatically (like a thermostat) | Most workloads - AWS recommended |
| **Step scaling** | Define step adjustments based on alarm breach magnitude | Fine-grained control at specific thresholds |
| **Simple scaling** | Single adjustment when alarm triggers, then cooldown | Legacy - not recommended |

**Target tracking predefined metrics**: `ASGAverageCPUUtilization`, `ASGAverageNetworkIn/Out`, `ALBRequestCountPerTarget`

### Scheduled Scaling

Scale at specific times using one-time or recurring (cron) schedules. Use for known traffic patterns (business hours, weekends, batch jobs). See [scheduled scaling in Amazon EC2 Auto Scaling](https://docs.aws.amazon.com/autoscaling/ec2/userguide/ec2-auto-scaling-scheduled-scaling.html).

### Predictive Scaling (ML-Based)

Uses machine learning to [forecast demand and pre-scale capacity](https://docs.aws.amazon.com/autoscaling/ec2/userguide/predictive-scaling-policy-overview.html).

- Requires minimum 24 hours of metric data (2 weeks recommended)
- Forecasts 48 hours ahead, updated every 6 hours
- Does NOT scale in automatically - combine with target tracking for scale-in
- Use for cyclical patterns and applications with long initialization times

**Best practice**: Combine predictive scaling (for scale-out) with target tracking (for scale-in and reactive adjustments).

> [↑ Back to Table of Contents](#table-of-contents)

---

## ASG and ELB Integration

See [attaching a load balancer to your ASG](https://docs.aws.amazon.com/autoscaling/ec2/userguide/attach-load-balancer-asg.html).

### Health Check Types

| Type | Default | Description |
| :--- | :--- | :--- |
| **EC2** | Yes | Checks instance status (running, not impaired) |
| **ELB** | No (must enable) | Uses load balancer health checks |

When ELB health checks are enabled, ASG replaces instances that fail load balancer health checks - not just EC2 status checks.

### Health Check Grace Period

Time ASG waits after an instance reaches InService state before checking health. Allows time for application startup.

### Instance Warmup

Time for new instances to warm up before contributing to CloudWatch metrics. Prevents premature scale-in decisions based on incomplete metrics.

| Concept | Applies To | Purpose |
| :--- | :--- | :--- |
| [Cooldown](https://docs.aws.amazon.com/autoscaling/ec2/userguide/ec2-auto-scaling-scaling-cooldowns.html) | Simple scaling only | Pause all scaling activities |
| Instance warmup | Target tracking, step scaling | Exclude warming instances from metrics |

> [↑ Back to Table of Contents](#table-of-contents)

---

## Multi-AZ Deployment

ASG [distributes instances across selected Availability Zones](https://docs.aws.amazon.com/autoscaling/ec2/userguide/ec2-auto-scaling-availability-zone-balanced.html) for high availability.

| Distribution Strategy | Behavior |
| :--- | :--- |
| **Balanced best effort** (default) | Tries to balance across AZs; falls back to healthy AZs on failure |
| **Balanced only** | Strictly balances; continues retrying failed AZ |

Configure by selecting subnets in multiple AZs during ASG creation.

> [↑ Back to Table of Contents](#table-of-contents)

---

## Shared Responsibility Model

### AWS Responsibilities

- ELB infrastructure, scaling, and high availability
- Auto Scaling service operation and API availability
- Physical infrastructure and network connectivity
- Managed service patches and updates

### Customer Responsibilities

- Security group rules for load balancers and instances
- SSL/TLS certificate management for HTTPS listeners
- Target health check configuration
- Scaling policy design and tuning
- Launch template/configuration maintenance (AMI updates, user data)
- Application-level health and readiness

> [↑ Back to Table of Contents](#table-of-contents)

---

## Best Practices

- Use target tracking scaling as the default - simplest and most effective for most workloads
- Combine predictive scaling with target tracking for production workloads with cyclical patterns
- Enable ELB health checks on ASGs to catch application-level failures (not just instance failures)
- Set appropriate health check grace periods to avoid premature instance termination during startup
- Use launch templates over launch configurations - they support versioning and new instance types
- Deploy across at least 2 AZs for high availability; ASG handles distribution automatically
- Use ALB for HTTP/HTTPS workloads with content-based routing; use NLB when you need static IPs or ultra-low latency
- Consider GWLB when integrating third-party security appliances (firewalls, IDS/IPS) into your traffic flow

> [↑ Back to Table of Contents](#table-of-contents)

---

## References

- [AWS Well-Architected Framework Definitions](https://docs.aws.amazon.com/wellarchitected/latest/framework/definitions.html) - official definitions for scalability, elasticity, and cloud concepts
- [Comparing ELB Types](https://aws.amazon.com/compare/the-difference-between-the-difference-between-application-network-and-gateway-load-balancing/) - AWS comparison of ALB vs NLB vs GWLB
- [How Elastic Load Balancing Works](https://docs.aws.amazon.com/elasticloadbalancing/latest/userguide/how-elastic-load-balancing-works.html) - ELB architecture and concepts
- [Step Scaling Policies](https://docs.aws.amazon.com/autoscaling/ec2/userguide/as-scaling-simple-step.html) - detailed step scaling configuration

> [↑ Back to Table of Contents](#table-of-contents)
