# Azure Storage Data Objects
- Blob
    - Unstructured data
- File
- Queue
    - Async message queueing between applications in Azure
- Table
- Disk

# Azure Storage Accounts
- Contains all Azure storage objects
- Unique namespace access to storage resources
- Highly Available
- Safe and Secure

## Type of Storage Accounts
- General-purpose v2
    - Storage for all Azure data objects
    - Only type that supports Data Lake Gen2 data
- General-purpose v1
    - Legacy account types for blobs, files, queues, and tables
    - Can upgrade to v2
- BlockBlobStorage
    - Premium-performance-only accounts
    - Designed for block blobs and append blobs
    - Recommended for scenarios requiring low storage latency, higher transaction rates, or where data objecst are smaller
- FileStorage
    - Premium-tier performance for files
    - Only stores files
    - Recommended for enterprise apps or high-performance scale apps
- BlobStorage
    - Legacy blob-only account
    - Use general-purpose v2 when possible
    - can be upgraded to v2

## Storage Account Endpoints
|          | Storage Account Name    | Storage Service Endpoint | Container Name | Object Name |
| -- | -- | -- | -- | -- |
| https:// | sa-blobstore-eastus-001 | .blob.core.windows.net | /demo | /az-104-outline.pdf |

## Storage Account Performance

- Standard
    - All storage account types
    - backup and disaster recovery data
    - Media
- Premium
    - Available for:
        - BlockBlob storage
        - FileStorage
        - GPv1 and GPv2 (unmanaged VHDs only)
    - Interactive
    - Analytics
    - AI/ML

NO CONVERSION BETWEEN STANDARD AND PREMIUM AFTER DEPLOYMENT

## Access Tiers
Hot
: Highest storage cost
: Lower access cost

Cool
: Lower storage cost
: Higher access cost
: 30 day minimum

Archive
: Lowest storage cost
: Highest access cost
: 180 day minimum
: Incures deletion cost if deleted before 180 days

## Replication Options
- Local-Redundant storage (LRS)
- Zone-Redundant storage (ZRS)
- Geo-Redundant storage (GRS)
- Geo-Zone-Redundant storage (GZRS)
- Read-Access Geo-Redundant Storage (RA-GRS)
- Read-Access Geo-Zone-Redundant Storage (RA-GZRS)

Important questions:
- What if an Azure datacenter fails?
- What if an Azure region fails?
- Do you need Read access?

### Redundancy in a Primary Region
LRS
: provides minimum level of redundancy
: three synchronous copies of a storage account
: data in a primary region
: housed in a single physical location (Availability Zone)

ZRS
: three synchronous copies in a primary region
: housed in one of three Availability Zone
: will survive entire datacenter outage

### Redundancy in a Secondary Region
GRS
: LRS in primary and secondary regions
: secondary region is read-only
: protects against loss of an entire region
: will not protect against single datacenter loss

GZRS
: ZRS in primary; LRS in secondary
: secondary region is read-only

### Read Access in a Secondary Region
- Read-Only without failover
- Applications can use secondary storage
- Available in Geo-redundant storage
    - Read-Access Geo-Redundant Storage (RA-GRS)
    - Read-Access Geo-Zone-Redundant Storage (RA-GZRS)
- Append -secondary to storage account name
    - https://\<StorageAccountName> ***-secondary***.blob.core.windows.net

### Azure Storage Durability and Availability Scenarios
# Will be on exam
| Outage Scenario | LRS | ZRS | GRS/RA-GRS | GZRS/RA-GZRS |
| --------------- | --- | --- | ---------- | ------------ |
| Datacenter node becomes unavailable | Yes | Yes | Yes | Yes |
| Entire datacenter becomes unavailable | No | Yes | Yes | Yes |
| Primary region-wide outage | No | No | Yes | Yes |
| Read access in secondary region when primary is unavailable | No | No | Yes (with RA-GRS) | Yes (with RA-GZRS) |
*Referenced from Microsoft at https://bit.ly/2FeFav5*

### Storage Account Supported Capabilities
|   | Data Objects | Performance Tiers | Access Tiers | Replication Options |
| - | - | - | - | - |
| General Purpose v2 | Blob, File, Table, Disk, Queue, & Data Lake Gen2 | Standard<br>Premium (Disk Only) | Hot, Cool. Archive | LRS, GRS, RA-GRS, ZRS, GZRS(preview), RA-GZRS(preview) |
| General Purpose v1 | Blob, File, Queue, Table, & Disk | Standard<br>Premium (Disk Only) | N/A | LRS, GRS, RA-GRS |
| BlockBlobStorage | Blob (block blobs and append blobs) | Premium | N/A | LRS, ZRS |
| FileStorage | File Only | Premium | N/A | LRS, ZRS |
| BlobStorage | Blob (block blobs and append blobs) | Standard | Hot, Cool, Archive | LRS, GRS, RA-GRS |
*Referenced from Microsoft at https://bit.ly/azstraccts*

# Configuring Access Control to Storage Accounts

## Data Storage Authorization
- Anonymous
    - Azure blobs only
- Authenticated
    - Shared Key authorization
        - App access storage with a storage access key
    - Shared Access Signatures (SAS)
        - limited delegated access to resources in storage account
    - Azure AD
        - RBAC to storage account

## Authorizing Access to Azure Storage Data

|      | Shared Key<br>(storage account Key) | Shared Access Signature (SAS) | Azure AD | Anonymous public read access |
| --- | --- | --- | --- | --- |
| Azure Blobs | Supported | Supported | Supported | Supported |
| Azure Files (SMB) | Supported | Not Supported | *Supported using Azure AD Domain Services ONLY | Not Supported |
| Azure Files (REST) | Supported | Supported | Not Supported | Not Supported |
| Azure Queues | Supported | Supported | Supported | Not Supported |
| Azure Tables | Supported | Supported | Not Supported | Not Supported |
*More information at https://bit.ly/2DHOGXa*

## Shared Access Key Authorization
- Two default keys
- Access to entire storage account
    - Basically root password to entire storage account
- Protect from unauthorized view
- Keys can be regenerated if compromised
- MS recommends using Azure AD
- Store in Azure Key Vault for increased security

## Shared Access Signatures (SAS)
- User Delegation
    - Azure AD and permissions
    - Blob storage only
- Service
    - Secured with storage account key
    - Delegates access to a resource in only one Azure storage services
        - Blob
        - Queue
        - Table
        - File
- Account
    - Secured with storage account key
    - Delegates accses to one or more storage services
    - Operations available in User Delegation and Service SAS are available with account SAS
    - Delegate access to read, write, and delete that are not permitted with service SAS
## Azure AD Authorization
- Supported for Blob and Queue storage
- Uses RBAC
- MS recommended approach

### Accessing Resources using Azure AD
- Data layer permissions
    - Grants access to storage
    - Does not grant access navigate through resources in Azure
- Management permissions
    - Grants access through resources

## Network Access Control
###Azure Storage Firewalls and Virtual Networks
- Layered security model
- Limit access by rules
    - IP addresses
    - IP ranges
    - Subnets in Azure vNets
- Requires authorization