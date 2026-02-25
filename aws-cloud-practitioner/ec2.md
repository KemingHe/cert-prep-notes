# Study Notes - Amazon EC2 (Elastic Compute Cloud)

> **Last Updated**: 2026-02-24 by Keming He

## What is EC2?

EC2 (Elastic Compute Cloud) is AWS's IaaS offering for renting virtual machines in the cloud. As IaaS, the customer manages everything from the OS up; AWS manages the physical infrastructure and hypervisor.

- **EC2 instances**: Virtual machines running your workloads
- **EBS / EFS**: Network-attached storage volumes
- **ELB** (Elastic Load Balancer): Distributes incoming traffic across multiple instances
- **ASG** (Auto Scaling Group): Automatically adjusts instance count based on load

An EC2 instance is fully defined by: AMI (OS) + instance size (CPU + RAM) + storage + security groups + IAM role + EC2 User Data (bootstrap script).

## Amazon Machine Image (AMI)

An AMI is a pre-configured template containing the OS and software stack used to launch EC2 instances.

- **Region-specific**: AMIs belong to one region; copy them to other regions as needed
- **Sources**: AWS-provided, AWS Marketplace, community-published, or custom (captured from an existing running instance)
- One AMI can launch many instances with identical configurations
- Custom AMIs let you pre-bake software so instances launch faster with less User Data work

## Instance Types and Naming

Instance names follow the pattern `[family][generation][options].[size]`, e.g., `t3.micro` or `c7gn.xlarge`.

| Category | Prefix | Best For |
| :--- | :--- | :--- |
| General purpose | `m`, `t` | Balanced compute, memory, and networking; `t` types offer burstable CPU |
| Compute optimized | `c` | Batch processing, media encoding, high-performance web servers |
| Memory optimized | `r`, `x` | In-memory databases, Apache Spark, real-time caching |
| Storage optimized | `i`, `d`, `h` | High IOPS workloads, data warehousing, distributed file systems |
| Accelerated computing | `p`, `g`, `f`, `inf`, `trn` | GPU workloads, FPGA, ML inference and training |
| HPC optimized | `hpc` | Tightly coupled high-performance computing with enhanced networking |

## Storage Options

| Type | Attachment | Persistence | Notes |
| :--- | :--- | :--- | :--- |
| **EBS** (Elastic Block Store) | Network-attached | Conditional (see notes below) | Single-AZ; attaches to one instance at a time |
| **EFS** (Elastic File System) | Network-attached | Survives independently | Shared across many instances and AZs; scales automatically; Linux only |
| **Instance Store** | Hardware-attached (physical disk on host) | Ephemeral - data lost on stop or terminate | Highest I/O performance; use for temp data, caches, or buffers only |

- Stopping an instance preserves all EBS and EFS data; private IP and IAM role are also retained
- Terminating an instance permanently deletes instance store data and the root EBS volume (by default); attached EBS data volumes persist unless configured with DeleteOnTermination

## Networking and IP Addressing

- **Public IP**: Assigned from AWS's pool on instance start; changes every time the instance is stopped and restarted
- **Private IP**: Assigned within the VPC at launch; persistent across stop/start cycles for the life of the instance
- **Elastic IP**: Static public IPv4 address you allocate to your account; survives stop/start and can be remapped to another instance; charged when not associated with a running instance; use sparingly

## Security Groups

Security groups are stateful firewalls controlling inbound and outbound traffic at the EC2 instance level.

- Contain **allow rules only** - traffic not explicitly permitted is implicitly denied
- **Stateful**: return traffic for allowed connections is automatically permitted in the opposite direction
- Rules reference **IP ranges (CIDR)** or **other security groups** (e.g., allow inbound only from the load balancer's security group)
- **VPC-scoped**: a security group belongs to one VPC; cannot span VPCs
- One instance can have multiple security groups attached; one security group can be attached to many instances
- Default behavior: all inbound blocked, all outbound allowed

### Key Ports

| Port | Protocol | Use |
| :--- | :--- | :--- |
| 22 | SSH / SFTP | Linux instance login; SFTP uses SSH as its transport |
| 80 | HTTP | Unencrypted web traffic |
| 443 | HTTPS | Encrypted web traffic |
| 21 | FTP | File transfer (unencrypted control channel) |
| 3389 | RDP | Windows instance login |

### Connecting to Instances

- **Linux / macOS / Windows 10+**: `ssh -i [key].pem ec2-user@[public-ip]`; first run `chmod 400 [key].pem` to protect the key file
- **Older Windows (before Windows 10)**: PuTTY with a `.ppk` converted key
- **EC2 Instance Connect**: browser-based SSH from the AWS Console; no local SSH client needed; works from any OS

### Troubleshooting

- **Connection timeout**: security group inbound rule is missing or misconfigured
- **Connection refused**: instance is reachable but no SSH daemon or application is listening on the port; try restarting the instance

## EC2 User Data (Bootstrap Scripts)

- Runs **once only** at first instance launch, as the `root` user (no `sudo` needed)
- Common uses: install packages, pull application code, write config files - any one-time setup before the app starts
- Longer User Data = longer first-boot time; bake frequently repeated setup into a custom AMI instead

## Purchasing Options

| Option | Commitment | Discount vs. On-Demand | Best For |
| :--- | :--- | :--- | :--- |
| **On-Demand** | None | Baseline price | Short, unpredictable, or uninterruptible workloads; no upfront payment |
| **Reserved Instances (RI)** | 1 or 3 years | Up to 72% | Steady-state workloads (e.g., production databases); locked to instance family + region |
| **Convertible RI** | 1 or 3 years | Up to 66% | Same as RI but allows changing instance family, OS, scope, and tenancy |
| **Savings Plans** | 1 or 3 years | Up to 72% | Commit to a $/hr usage amount; flexible across instance size, OS, and tenancy within a family/region; AWS-recommended over RIs |
| **Spot Instances** | None | Up to 90% | Fault-tolerant, stateless, batch, or distributed workloads; AWS reclaims with a **2-minute warning** |
| **Dedicated Hosts** | On-Demand or 1/3 yr RI | Varies | Compliance needs or BYOL server-bound software licenses; most expensive option; full visibility and control over socket/core placement |
| **Dedicated Instances** | On-Demand | Varies | Single-tenant hardware; no control over host placement; may share the same host with other instances in the same account |
| **Capacity Reservation** | None | 0% (no discount) | Reserve On-Demand capacity in a specific AZ for any duration; billed at On-Demand rates whether or not instances are running |

Key rules:

- More upfront payment = larger discount (All Upfront > Partial Upfront > No Upfront)
- RI terms are fixed at 1 year or 3 years - no 2-year option
- Standard RIs can be listed and sold in the RI Marketplace; Convertible RIs cannot
- Spot usage beyond a Savings Plan commitment is billed at On-Demand rates

## Shared Responsibility Model

### AWS Responsibilities

- Physical data center and hardware security, maintenance, and replacement
- Hypervisor and host OS security and patching
- Physical isolation between tenants (including Dedicated Hosts)
- Infrastructure SLA compliance and availability

### Customer Responsibilities

- Security group rules and network ACL configuration
- Guest OS patching and updates (EC2 is IaaS - you own the OS)
- Application software security, updates, and configuration
- IAM role and policy assignments to instances
- Data encryption at rest (EBS/EFS) and in transit

## Best Practices

- Attach IAM roles to EC2 instances for AWS API access - never store access keys directly on an instance
- Maintain a dedicated security group for SSH/RDP access with restricted source IPs; keep it separate from application traffic rules
- Use EC2 User Data for repeatable bootstrapping; bake frequently reused setup into a custom AMI to reduce boot time
- Prefer Savings Plans over Reserved Instances for most workloads - more flexible at similar discount levels
- Design Spot Instance workloads to checkpoint state and handle the 2-minute interruption warning gracefully
- Use DNS names or load balancers for stable endpoints rather than Elastic IPs to avoid idle IP charges

---

> Study Notes Template v1.0.0 - KemingHe/cert-prep-notes
