# Study Notes - EC2 Storage Options

> **Last Updated**: 2026-03-16 by Keming He

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
  - [References](#references)

## Storage Overview

EC2 offers three storage types with distinct characteristics:

| Type | Attachment | Persistence | Max IOPS | Best For |
| :--- | :--- | :--- | :--- | :--- |
| [**EBS**](#ebs-elastic-block-store) | Network | Survives stop; root volume deleted on terminate by default | [256K (io2 Block Express)](https://docs.aws.amazon.com/ebs/latest/userguide/provisioned-iops.html) | General block storage, databases |
| [**EFS**](#efs-elastic-file-system) | Network | Survives independently | Elastic | Shared Linux file systems |
| [**Instance Store**](#ec2-instance-store) | Physical (host disk) | Ephemeral - lost on stop, hibernate, or terminate | [7M+](https://docs.aws.amazon.com/ec2/latest/instancetypes/so.html) | Caches, buffers, temp data |

**IOPS** (Input/Output Operations Per Second) measures storage read/write performance - higher values mean faster disk operations.

> [↑ Back to Table of Contents](#table-of-contents)

---

## EBS (Elastic Block Store)

[EBS volumes](https://docs.aws.amazon.com/ebs/latest/userguide/ebs-volumes.html) are network-attached block storage drives for EC2 instances.

- **AZ-bound**: A volume exists in one Availability Zone; [cannot attach to instances in other AZs](https://docs.aws.amazon.com/ebs/latest/userguide/ebs-attaching-volume.html)
- **Network drive**: Slight latency compared to physical disks; can detach and reattach like a "network USB stick"
- **Provisioned capacity billing**: [Charged for allocated size](https://aws.amazon.com/ebs/pricing/) (e.g., 100 GB), not actual usage
- **Unattached volumes**: Valid state; useful for keeping data without a running instance

### Attachment Rules

| Configuration | Supported |
| :--- | :--- |
| One EBS to one instance | Yes (standard) |
| Multiple EBS to one instance | Yes (root + data volumes) |
| One EBS to multiple instances | Only io1/io2 with [Multi-Attach](https://docs.aws.amazon.com/ebs/latest/userguide/ebs-volumes-multi.html) (up to 16 Nitro instances, same AZ) |

### Delete on Termination

| Volume Type | Default Behavior |
| :--- | :--- |
| Root volume | [Deleted on termination](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/preserving-volumes-on-termination.html) |
| Data volumes attached via Console or after launch | [Preserved on termination](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/preserving-volumes-on-termination.html) |
| Data volumes attached at launch via CLI | [Deleted on termination](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/preserving-volumes-on-termination.html) |

Configure via the `DeleteOnTermination` attribute when launching or in the console.

> [↑ Back to Table of Contents](#table-of-contents)

---

## EBS Snapshots

[Snapshots](https://docs.aws.amazon.com/ebs/latest/userguide/ebs-snapshots.html) are point-in-time backups of EBS volumes stored in S3 (managed by AWS, not visible in your S3 console).

- **Region-bound**: [Stored redundantly across AZs](https://docs.aws.amazon.com/ebs/latest/userguide/ebs-snapshots.html) within a region
- **Cross-AZ transfer**: Snapshot in us-east-1a, restore in us-east-1b - effectively moves data across AZs
- **Cross-region copy**: [Copy snapshots to other regions](https://docs.aws.amazon.com/ebs/latest/userguide/ebs-copy-snapshot.html) for disaster recovery
- **Detach first**: Not required but [recommended before taking a snapshot](https://docs.aws.amazon.com/prescriptive-guidance/latest/backup-recovery/new-ebs-volume-backups.html) for data consistency

### Snapshot Features

| Feature | Description |
| :--- | :--- |
| **[Archive tier](https://docs.aws.amazon.com/ebs/latest/userguide/snapshot-archive-considerations.html)** | Up to 75% cheaper; up to 72 hours to restore; 90-day minimum storage |
| **[Recycle bin](https://docs.aws.amazon.com/ebs/latest/userguide/recycle-bin-create-rule.html)** | Retain deleted snapshots 1 day to 1 year; protects against accidental deletion |
| **Create volume** | Restore to a new volume in any AZ within the region |

> [↑ Back to Table of Contents](#table-of-contents)

---

## AMI (Amazon Machine Image)

An [AMI](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/AMIs.html) is a template containing OS, software, and configuration for launching EC2 instances.

- **Region-bound**: [AMIs belong to one region](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/CopyingAMIs.html); copy to other regions as needed
- **Sources**: [AWS-provided, AWS Marketplace, community-published, or custom](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/sharing-amis.html)

### AMI Creation Workflow

1. Launch and customize an EC2 instance
2. Stop the instance (ensures data consistency)
3. Create AMI (automatically creates EBS snapshots)
4. Launch new instances from the AMI

### EC2 Image Builder

[EC2 Image Builder](https://docs.aws.amazon.com/imagebuilder/latest/userguide/what-is-image-builder.html) automates AMI creation, maintenance, validation, and testing.

- **Free service**: [Pay only for underlying EC2 instances and storage](https://docs.aws.amazon.com/imagebuilder/latest/userguide/what-is-image-builder.html) during builds
- **Scheduling**: Run on schedule or trigger on software updates
- **Multi-region distribution**: [Distribute validated AMIs across regions](https://docs.aws.amazon.com/imagebuilder/latest/userguide/what-is-image-builder.html) automatically

**Pipeline steps**:

1. Create builder EC2 instance with specified software and configuration
2. Generate AMI from the builder instance
3. Launch test instance and run validation tests
4. Distribute passing AMI to target regions

> [↑ Back to Table of Contents](#table-of-contents)

---

## EC2 Instance Store

[Instance store](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/InstanceStorage.html) provides temporary block storage on disks physically attached to the host machine.

- **Highest I/O performance**: [Millions of IOPS](https://docs.aws.amazon.com/ec2/latest/instancetypes/so.html) (vs. hundreds of thousands for EBS io2)
- **Ephemeral**: Data is [cryptographically erased](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/instance-store-lifetime.html) on stop, hibernate, terminate, or hardware failure
- **No extra cost**: [Included with instance types](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/InstanceStorage.html) that support it

### Performance Comparison

| Storage | Max IOPS | Latency |
| :--- | :--- | :--- |
| [EBS gp3](https://docs.aws.amazon.com/ebs/latest/userguide/general-purpose.html) | 80,000 | Single-digit ms |
| [EBS io2 Block Express](https://docs.aws.amazon.com/ebs/latest/userguide/provisioned-iops.html) | 256,000 | Sub-ms |
| [Instance Store (i3.16xlarge)](https://docs.aws.amazon.com/ec2/latest/instancetypes/so.html) | 3.3M read / 1.44M write | Sub-ms |
| [Instance Store (i7i.48xlarge)](https://docs.aws.amazon.com/ec2/latest/instancetypes/so.html) | 7.2M read / 3.96M write | Sub-ms |

### Use Cases

- Buffers and caches (regenerable data)
- Scratch space for processing jobs
- Temporary session data
- Replicated data across instance fleets

**Warning**: EBS-backed instances can recover on hardware failure; [instance store-backed instances lose all instance store data](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-instance-recover.html) during recovery.

> [↑ Back to Table of Contents](#table-of-contents)

---

## EFS (Elastic File System)

[EFS](https://docs.aws.amazon.com/efs/latest/ug/whatisefs.html) is a managed NFS (Network File System) mountable by [up to 25,000 EC2 instances](https://docs.aws.amazon.com/efs/latest/ug/limits.html) simultaneously.

- **Linux only**: [Not compatible with Windows EC2 instances](https://docs.aws.amazon.com/efs/latest/ug/limits.html); use FSx for Windows workloads
- **Multi-AZ**: [Regional file systems span all AZs](https://docs.aws.amazon.com/efs/latest/ug/features.html); One Zone option available at lower cost
- **Pay-per-use**: No capacity provisioning; [scales automatically](https://docs.aws.amazon.com/efs/latest/ug/features.html)
- **Cost**: [~3-4x EBS gp3 pricing](https://aws.amazon.com/efs/pricing/), offset by no capacity planning and shared access

### Mount Targets

- [One mount target per AZ](https://docs.aws.amazon.com/efs/latest/ug/limits.html) (even with multiple subnets)
- All instances in an AZ share the same mount target
- Cross-AZ access incurs [data transfer charges ($0.01/GB)](https://aws.amazon.com/efs/pricing/)
- Each mount target can have [up to 5 security groups](https://docs.aws.amazon.com/efs/latest/ug/limits.html)

### Storage Classes

| Class | Cost vs Standard | Access Charge | Use Case |
| :--- | :--- | :--- | :--- |
| Standard | Baseline | None | Frequently accessed files |
| [Infrequent Access (IA)](https://aws.amazon.com/efs/pricing/) | Up to 94% lower | Per-read charge | Infrequently accessed files |

Configure [lifecycle policies](https://docs.aws.amazon.com/efs/latest/ug/lifecycle-management-efs.html) to automatically move files to IA based on access patterns - transparent to applications.

> [↑ Back to Table of Contents](#table-of-contents)

---

## Amazon FSx

[FSx](https://docs.aws.amazon.com/fsx/) provides fully managed third-party high-performance file systems.

| File System | Protocol | Best For |
| :--- | :--- | :--- |
| **[FSx for Lustre](https://docs.aws.amazon.com/fsx/latest/LustreGuide/what-is.html)** | POSIX | HPC, ML training, video processing |
| **[FSx for Windows File Server](https://docs.aws.amazon.com/fsx/latest/WindowsGuide/what-is.html)** | SMB, NTFS | Windows workloads, Active Directory environments |
| **[FSx for NetApp ONTAP](https://docs.aws.amazon.com/fsx/latest/ONTAPGuide/what-is-fsx-ontap.html)** | NFS, SMB, iSCSI | Enterprise NAS, multi-protocol access |
| **[FSx for OpenZFS](https://docs.aws.amazon.com/fsx/latest/OpenZFSGuide/what-is-fsx.html)** | NFS | Unix/Linux ZFS workloads, snapshots |

### FSx for Windows File Server

- Fully managed Windows-native shared file system
- [Supports SMB protocol and Windows NTFS](https://docs.aws.amazon.com/fsx/latest/WindowsGuide/what-is.html)
- [Integrates with Microsoft Active Directory](https://docs.aws.amazon.com/fsx/latest/WindowsGuide/self-managed-AD.html) (AWS Managed or self-managed)
- [Accessible from AWS and on-premises infrastructure](https://docs.aws.amazon.com/fsx/latest/WindowsGuide/what-is.html)
- Region-bound; supports [cross-region backup via AWS Backup](https://docs.aws.amazon.com/fsx/latest/WindowsGuide/using-backups.html)

### FSx for Lustre

- High-performance parallel file system for compute-intensive workloads
- **Performance**: [TBps throughput, millions of IOPS, sub-ms latency (SSD)](https://docs.aws.amazon.com/fsx/latest/LustreGuide/what-is.html) or single-digit ms (HDD)
- **S3 integration**: [Data Repository Associations](https://docs.aws.amazon.com/fsx/latest/LustreGuide/create-dra-linked-data-repo.html) enable bidirectional sync with S3
- **Naming origin**: [Linux + Cluster](https://lustre.org/about)
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

---

## References

- [Amazon EBS User Guide](https://docs.aws.amazon.com/ebs/latest/userguide/)
- [Amazon EC2 Instance Types](https://docs.aws.amazon.com/ec2/latest/instancetypes/)
- [Amazon EFS User Guide](https://docs.aws.amazon.com/efs/latest/ug/)
- [Amazon FSx Documentation](https://docs.aws.amazon.com/fsx/)
- [AWS Shared Responsibility Model](https://aws.amazon.com/compliance/shared-responsibility-model/)

> [↑ Back to Table of Contents](#table-of-contents)
