## The Exams

I recently went through the Microsoft Azure Administrator [AZ-104](https://learn.microsoft.com/en-us/certifications/exams/az-104) and Designing Microsoft Azure Infrastructure Solutions [AZ-305](https://learn.microsoft.com/en-us/certifications/exams/az-305) exams to unlock the [Azure Solutions Architect Expert](https://learn.microsoft.com/en-us/certifications/azure-solutions-architect/) certification. There is a significant amount of overlap in the content between the two and while I have a number of years experience working with Azure in various organisations, I definitely still needed to study and refresh myself on some of the concepts I was less familiar with.

## Study Materials
To start with, I cannot recommend [John Savill](https://www.youtube.com/@NTFAQGuy)'s Youtube channel enough for all things Azure Infrastructure. It is a pleasure to study with his content. His [AZ-305 Study Cram](https://www.youtube.com/watch?v=vq9LuCM4YP4) video is no exception with nearly 200k views! Not to mention his [AZ-104 Study Playlist](https://www.youtube.com/playlist?list=PLlVtbbG169nGlGPWs9xaLKT1KfwqREHbs) and [AZ-305 Study Playlist](https://www.youtube.com/playlist?list=PLlVtbbG169nHSnaP4ae33yQUI3zcmP5nP).

Also highly recommend the Microsoft Learn study paths which are a good mix of theory and pseudo practical:
- [AZ-104](https://learn.microsoft.com/en-us/training/paths/az-104-administrator-prerequisites/)
- [AZ-305](https://learn.microsoft.com/en-us/training/paths/microsoft-azure-architect-design-prerequisites/)

Lastly, the MicrosoftLearning labs on Github are excellent to lock in the practical skills:
- [AZ-104](https://github.com/MicrosoftLearning/AZ-104-MicrosoftAzureAdministrator)
- [AZ-305](https://github.com/MicrosoftLearning/AZ-305-DesigningMicrosoftAzureInfrastructureSolutions)

> Note: The below notes are still roughly in my own mental language and may be difficult to consume for readers, but I need to get an MVP out or I'll never publish this post. I'll tidy it up over time (hopefully).

## Tips & Tricks
As with all Microsoft exams, often they present more than one correct answer but they want to see the 'most correct' answer or the 'Microsoft correct' answer. In this exam, the main thing I noticed was a recurring theme around cost management which meant there was often at least two valid answers, but one was more cost effective than the others.

This means you need to be aware of the SKUs for various services and which SKUs provide which level of functionality. For example, looking at a few features from Azure Firewall:

| Feature | Basic | Standard | Premium |
| --- | --- | --- | --- |
| FQDN Filtering (www.google.com) | X | X | X |
| Network Filtering (IP:Protocol) |  | X | X |
| Network Address Translation | X | X | X |
| Threat Intelligence Filtering |  | X | X |
| Web Content Filtering (Categories) |  | X | X |
| TLS Termination/Inspection |  |  | X |
| Intrusion Detection/Prevention |  |  | X |
| URL Filtering (www.google.com/news) |  |  | X |
| App/User Awareness |  |  | X |

## My Study Notes
####  Identity
- Management Groups
  - Policy | RBAC | Budget (inherits down)
- Subscriptions
  - Policy | RBAC | Budget (inherits down)
  - Be aware of resource limits, consider how many Subscriptions are right for your requirements
- Resource Groups
  - Policy | RBAC | Budget (inherits down)
  - Cannot nest Resource Groups
  - Resource group region is purely metadata, does not impact the location of resources within it

#### Policy
- Policies can be grouped into Initiatives and applied to a scope

#### Resource Tags
- Unstructured metadata
- Not inherited by default, could be made so with Azure Policy

#### RBAC
- Identity + Scope + Role + RoleDefinition
  - Identity is who is getting the access
  - Scope is what they are getting access to
  - Role is what access they will be assigned
  - RoleDefintion is what they can do with that access
- Roles can have Management Plane Actions and Data Plane Actions defined e.g.
  - Management Plane: `Owner`
  - Data Plane: `Blob Data Contributor`

#### PIM
- Privileged Identity Management - Just In Time requires **Azure AD Premium P2**
- Access Reviews requires Identity Governance Add-On or **Azure AD Premium P2**
  - Roles | Groups | Apps
- Azure BluePrints, made up of two parts:
  - Artefacts: 
    - Resource Groups + ARM Templates + RBAC + Policy
    - Can be stored at MG or RG level, and is available down the hierarchy
  - Config: Apply | Don't Lock (allow changes) | Do Not Delete | Read Only

#### Azure Active Directory
- **Azure AD Connect** (runs on-prem)
- **Azure AD Cloud Sync** (runs in the Cloud)
- **Hash the Hash**: Password sync
- **Azure Identity Protection**: Dark Web scanning for Password Hashes (**AAD P2**)
- **Seamless Sign-On**: Nice UX for logging into a computer that is on a network that can talk to a Domain Controller
- **Azure AD Domain Services**: Slim version of AD to allow Kerberos and Radius in the cloud (primarily for when you don't have an on-prem domain)

> AAD supports OAUTH2, SAML, WS-FED
> ADDS managed AD in a VNET supports LDAP, RADIUS

#### B2B AAD
- B2B invite > stub account (MS, Gmail, SAML, Email, SMS with an OTP)
  - User Flows can be configured for sign-in requirements / processes
- MFA happens on the host AAD, not the users 'home' IDP

#### B2C AAD
- Local & Social Accounts (FB, Google, Github etc) for Customers / Consumers
- Hide the URL with Azure Front Door
- Onboarding flows and UX for Customers

#### Conditional Access
- Specify conditions required to sign-in (AAD P1)
- Assign to users | groups | roles | applications | azure portal
- User risk | Sign-in risk | Locations | IP Locations | Device state > Allow or Deny

#### Application Identity | Managed Identity
- System assigned Service Principal -  tied to an Azure Resource, 1-1 (Delete the resource, the SP is deleted too)
- User assigned Service Principal - 1-n (Multiple resources can use the same Managed Identity)

#### Secrets with Azure Key Vault
- Storing and retrieving secrets with RBAC
  - 'Vault Access Policy' legacy Key Vault that was not integrated into Azure
  - 'Azure RBAC Policy' modern Key Vault with Azure integration for Azure Identity
- Managed Identity can be given GET permission to secret(s)
- Some Azure Services have abstracted Key Vault integration e.g. **AKS Key Vault CSI Driver**
- Key Vaults can be attached to a **Paired Region**, if the Primary Region goes offline the Paired Region is available but will be **Read Only**

#### Monitoring
- Diagnostic Settings are consistent across most layers and services of Azure
  - Log Analytics | Event Hub | Storage Account for logs

#### Azure Monitor
- Alert Rules > Action Group > SMS | Email | API | Function

#### Business Continuity
- Availability Sets | Availability Zones | Cross-Region
- Azure Site Recovery (Hot Cold) disk driver replicates blocks to another Region
- App-level replication (Hot Warm): requires running resource in both Regions

### Load Balancing for Business Continuity
- Might need to consider pairing a local solution with a global solution to achieve a single entry point for Users with multiple Regions behind it
- Layer 4 Standard Load Balancer **LOCAL**
- Layer 7 Application Gateway **LOCAL**
- Layer 7 Azure Front Door **GLOBAL**
  - Shortest path routing, TSL offloading, path based routing etc
- Traffic Manager (DNS) **GLOBAL**
  - Shortest path, lowest latency

### Backup & Restore - Azure Backup Service
- Replication never replaces Backups
- Focus on what you need to restore/recover
- **Azure Backup**
  - **Backup Vaults** (legacy)
    - Supports: Azure DB | Azure Blobs | Azure Disks
    - Data is copied into the Vault in some cases (VM Backups)
    - Can act as an Orchestrator for things like Blob snapshots
  - **Recovery Services Vaults** (modern)
    - Supports: Azure VM | SQL in Azure VM | Azure Files | SAP HANA in Azure VM | Azure Backup Server | Azure Backup Agent | DPM
    - Supports VM instant snapshots as well as stored backups
  - Vaults can be GRS | RA-GRS

### Design Data Solutions
- DP-900 Azure Data Fundamentals similar

### Types of Data
- **Structured**: Tables with a schema
- **Semi-Structured**: JSON or XML documents, self describing, no schema
- **Unstructured**: Documents, Media etc (Blob)
  - Storage Account: 
    - Blob - Block | Page (Read-Write anywhere in the file eg VHD) | Append (Write to the end eg Logs)
    - Files - SMB / NFS
    - Queues - first in, first out
  - Types:
    - General Purpose v2 Transaction Optimised | Hot | Cool | Archive
    - Premium: tied to a specific service eg Block, Files, PageBlob
      - *Premium Performance gives the best performance but cannot be GRS/Global*
    - Standard Performance allows GRS/Global
  - Multiple Storage Accounts decision making based on:
    - Encryption requirements, performance, data soveriegnty etc
  - Premium Storage Accounts do no support GRS, only LRS and ZRS

### Azure Files
- SMB can integrate with ADDS or AAD DS
- Azure File Sync maintains ACLS with ADDS aware
  - Azure File Sync can be used for Tiering i.e. 80% full, start to offload to the Cloud
- Premium / Transaction Optimised / Hot / Cool
  - Premium is the highest cost to store, the lowest/included cost for transactions
  - Cool is cheap to store, expensive to transact
- NetApp Files are more performant than Azure Files Premium, more expensive

### Managed Disks
- Page Blob on a Storage Account under the hood
- STD HDD / STD SSD / Premium SSD / Ultra Disk
- Ultra Disks have Capacity / Iops / Throughput dials
- STD and Premium SSDs have bursting ability (30 minutes based on credits accrued)

### Storage Account Encryption
- MMK or BYOK - BYOK is stored in Azure Key Vault
  - Updated a version of a key will automatically get picked up and re-encrypted

### Storage Security
- Access Keys (don't use them)
- Shared Access Signature (SAS) - URL with a token
  - Account SAS
  - Service SAS
  - Signed by the access key, SAS will no longer work if Access Key is disabled
- RBAC Data Plane (Blob, Files, Queues)
  - RBAC is not supported for Tables
- Secure Transfer (HTTPS) required
  - Enforces SMB 3

### SQL 
- Azure SQL Database | PaaS | 100TB max
  - Built in Auto Scale
  - Tiers
    - General Purpose (some downtime, outage another 'node' would attach to storage and start up)
    - Business Critical (Primary and Secondary nodes in an Availability Group)
    - Hyperscale (Sharded data, still a single Primary node, includes a Log Server and Page Servers - distributes the request across the Page Servers / Data shards)

- Azure SQL Managed Instance
  - PaaS in a VNET, better compatibility

- Azure SQL in IaaS VM
  - VM Extension, automated backups, automated patching
  - Full compatibility/features

- SQL Scaling
  - Scaling up is easy, bigger SKU
  - Scaling out is challenging, Read Replicas possible
  - Elastic Pool - shared resources, shared DTU, shared storage, flexible for dynamic workloads

My final comment is to say, don't be afraid to book in your exams, just get the ball rolling - failing an exam is not an end of the world event. Learn from it and tackle it again after doing some more study. PS. Nobody but you and Microsoft need to know you needed a second try ;) 

Good Luck (not that you'll need it) & Happy Clouding,

### Contact

- [LinkedIn](https://www.linkedin.com/in/adamcybersec/)<br>
- [GitHub](https://github.com/adamcybersec/)<br>
- [Email](mailto:github@adamcybersec.com)

