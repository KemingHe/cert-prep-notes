# Study Notes - EC2 Storage Options

> **Last Updated**: 2026-02-25 by Keming He

## Table of Contents

- [Study Notes - EC2 Storage Options](#study-notes---ec2-storage-options)
  - [Table of Contents](#table-of-contents)
  - [Storage Overview](#storage-overview)
  - [EBS (Elastic Block Store)](#ebs-elastic-block-store)
    - [Attachment Rules](#attachment-rules)
    - [Delete on Termination](#delete-on-termination)
  - [EBS Snapshots](#ebs-snapshots)
    - [Snapshot Features](#snapshot-features)
  - [AMI (Amazon Machine Image)](#ami-amazon-machine-image)
    - [AMI Creation Workflow](#ami-creation-workflow)
    - [EC2 Image Builder](#ec2-image-builder)
  - [EC2 Instance Store](#ec2-instance-store)
    - [Performance Comparison](#performance-comparison)
    - [Use Cases](#use-cases)
  - [EFS (Elastic File System)](#efs-elastic-file-system)
    - [Mount Targets](#mount-targets)
    - [Storage Classes](#storage-classes)
  - [Amazon FSx](#amazon-fsx)
    - [FSx for Windows File Server](#fsx-for-windows-file-server)
    - [FSx for Lustre](#fsx-for-lustre)
  - [Shared Responsibility Model](#shared-responsibility-model)
    - [AWS Responsibilities](#aws-responsibilities)
    - [Customer Responsibilities](#customer-responsibilities)
  - [Best Practices](#best-practices)

## Storage Overview

EC2 offers three storage types with distinct characteristics:

| Type | Attachment | Persistence | Max IOPS | Best For |
| :--- | :--- | :--- | :--- | :--- |
| [**EBS**](#ebs-elastic-block-store) | Network | Survives stop; root volume deleted on terminate by default | 256K (io2 Block Express) | General block storage, databases |
| [**EFS**](#efs-elastic-file-system) | Network | Survives independently | Elastic | Shared Linux file systems |
| [**Instance Store**](#ec2-instance-store) | Physical (host disk) | Ephemeral - lost on stop, hibernate, or terminate | 7M+ | Caches, buffers, temp data |

**IOPS** (Input/Output Operations Per Second) measures storage read/write performance - higher values mean faster disk operations.

> [↑ Back to Table of Contents](#table-of-contents)

---

## EBS (Elastic Block Store)

EBS volumes are network-attached block storage drives for EC2 instances.

- **AZ-bound**: A volume exists in one Availability Zone; cannot attach to instances in other AZs
- **Network drive**: Slight latency compared to physical disks; can detach and reattach like a "network USB stick"
- **Provisioned capacity billing**: Charged for allocated size (e.g., 100 GB), not actual usage
- **Unattached volumes**: Valid state; useful for keeping data without a running instance

### Attachment Rules

| Configuration | Supported |
| :--- | :--- |
| One EBS to one instance | Yes (standard) |
| Multiple EBS to one instance | Yes (root + data volumes) |
| One EBS to multiple instances | Only io1/io2 with _Multi-Attach_ (up to 16 Nitro instances, same AZ) |

### Delete on Termination

| Volume Type | Default Behavior |
| :--- | :--- |
| Root volume | Deleted on termination |
| Additional data volumes | Preserved on termination |

Configure via the `DeleteOnTermination` attribute when launching or in the console.

> [↑ Back to Table of Contents](#table-of-contents)

---

## EBS Snapshots

Snapshots are point-in-time backups of EBS volumes stored in S3 (managed by AWS, not visible in your S3 console).

- **Region-bound**: Stored redundantly across AZs within a region
- **Cross-AZ transfer**: Snapshot in us-east-1a, restore in us-east-1b - effectively moves data across AZs
- **Cross-region copy**: Copy snapshots to other regions for disaster recovery
- **Detach first**: Not required but recommended before taking a snapshot for data consistency

### Snapshot Features

| Feature | Description |
| :--- | :--- |
| **Archive tier** | Up to 75% cheaper; up to 72 hours to restore; 90-day minimum storage |
| **Recycle bin** | Retain deleted snapshots 1 day to 1 year; protects against accidental deletion |
| **Create volume** | Restore to a new volume in any AZ within the region |

> [↑ Back to Table of Contents](#table-of-contents)

---

## AMI (Amazon Machine Image)

An AMI is a template containing OS, software, and configuration for launching EC2 instances.

- **Region-bound**: AMIs belong to one region; copy to other regions as needed
- **Sources**: AWS-provided, AWS Marketplace, community-published, or custom

### AMI Creation Workflow

1. Launch and customize an EC2 instance
2. Stop the instance (ensures data consistency)
3. Create AMI (automatically creates EBS snapshots)
4. Launch new instances from the AMI

### EC2 Image Builder

Automates AMI creation, maintenance, validation, and testing.

- **Free service**: Pay only for underlying EC2 instances and storage during builds
- **Scheduling**: Run on schedule or trigger on software updates
- **Multi-region distribution**: Distribute validated AMIs across regions automatically

**Pipeline steps**:

1. Create builder EC2 instance with specified software and configuration
2. Generate AMI from the builder instance
3. Launch test instance and run validation tests
4. Distribute passing AMI to target regions

> [↑ Back to Table of Contents](#table-of-contents)

---

## EC2 Instance Store

Instance store provides temporary block storage on disks physically attached to the host machine.

- **Highest I/O performance**: Millions of IOPS (vs. hundreds of thousands for EBS io2)
- **Ephemeral**: Data is cryptographically erased on stop, hibernate, terminate, or hardware failure
- **No extra cost**: Included with instance types that support it

### Performance Comparison

| Storage | Max IOPS | Latency |
| :--- | :--- | :--- |
| EBS gp3 | 16,000 | Single-digit ms |
| EBS io2 Block Express | 256,000 | Sub-ms |
| Instance Store (i3.16xlarge) | 3.3M read / 1.4M write | Sub-ms |
| Instance Store (i7i.48xlarge) | 7.2M read / 4M write | Sub-ms |

### Use Cases

- Buffers and caches (regenerable data)
- Scratch space for processing jobs
- Temporary session data
- Replicated data across instance fleets

**Warning**: EBS-backed instances can migrate on hardware failure; instance store-backed instances are terminated with complete data loss.

> [↑ Back to Table of Contents](#table-of-contents)

---

## EFS (Elastic File System)

EFS is a managed NFS (Network File System) mountable by hundreds of EC2 instances simultaneously.

- **Linux only**: Not compatible with Windows EC2 instances; use FSx for Windows workloads
- **Multi-AZ**: Regional file systems span all AZs; One Zone option available at lower cost
- **Pay-per-use**: No capacity provisioning; scales automatically
- **Cost**: ~3-4x EBS gp3 pricing, offset by no capacity planning and shared access

### Mount Targets

- One mount target per AZ (even with multiple subnets)
- All instances in an AZ share the same mount target
- Cross-AZ access incurs data transfer charges ($0.01/GB)
- Each mount target can have up to 5 security groups

### Storage Classes

| Class | Cost vs Standard | Access Charge | Use Case |
| :--- | :--- | :--- | :--- |
| Standard | Baseline | None | Frequently accessed files |
| Infrequent Access (IA) | Up to 94% lower | Per-read charge | Infrequently accessed files |

Configure lifecycle policies to automatically move files to IA based on access patterns - transparent to applications.

> [↑ Back to Table of Contents](#table-of-contents)

---

## Amazon FSx

FSx provides fully managed third-party high-performance file systems.

| File System | Protocol | Best For |
| :--- | :--- | :--- |
| **FSx for Lustre** | POSIX | HPC, ML training, video processing |
| **FSx for Windows File Server** | SMB, NTFS | Windows workloads, Active Directory environments |
| **FSx for NetApp ONTAP** | NFS, SMB, iSCSI | Enterprise NAS, multi-protocol access |
| **FSx for OpenZFS** | NFS | Unix/Linux ZFS workloads, snapshots |

### FSx for Windows File Server

- Fully managed Windows-native shared file system
- Supports SMB protocol and Windows NTFS
- Integrates with Microsoft Active Directory (AWS Managed or self-managed)
- Accessible from AWS and on-premises infrastructure
- Region-bound; supports cross-region backup via AWS Backup

### FSx for Lustre

- High-performance parallel file system for compute-intensive workloads
- **Performance**: TBps throughput, millions of IOPS, sub-ms latency (SSD) or single-digit ms (HDD)
- **S3 integration**: Data Repository Associations enable bidirectional sync with S3
- **Naming origin**: Linux + Cluster
- Region-bound; use S3 cross-region replication for multi-region strategies

> [↑ Back to Table of Contents](#table-of-contents)

---

## Shared Responsibility Model

### AWS Responsibilities

- Infrastructure maintenance and physical security
- Data replication for EBS volumes and EFS drives
- Replacing faulty hardware
- Ensuring data confidentiality at rest (encryption infrastructure)

### Customer Responsibilities

- Configuring backups and snapshot schedules
- Enabling and managing encryption settings
- Data content and access control
- Understanding and accepting Instance Store hardware failure risks
- Security group configuration for EFS mount targets

> [↑ Back to Table of Contents](#table-of-contents)

---

## Best Practices

- **Choose storage by workload**: EBS for single-instance block storage, EFS for shared Linux file systems, Instance Store only for ephemeral high-IOPS needs
- **Enable snapshots**: Automate EBS snapshots for critical volumes; use lifecycle policies for retention
- **Use lifecycle policies**: Configure EFS-IA transitions to reduce costs for infrequently accessed data
- **Plan for failure**: Never store irreplaceable data on Instance Store; replicate critical data across AZs
- **Right-size EBS**: You pay for provisioned capacity; start smaller and resize as needed
- **Secure mount targets**: Apply security groups to EFS mount targets; restrict NFS port 2049 access
- **Consider FSx for specialized workloads**: Windows environments benefit from FSx for Windows; HPC benefits from FSx for Lustre

> [↑ Back to Table of Contents](#table-of-contents)
