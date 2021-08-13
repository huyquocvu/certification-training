<style>
    .important-boxed {
        background: #D9E5D6;
        color: black;
        font-weight: bold;
        border: 5px solid #E63946;
        margin: 0px auto;
        width: auto;
        padding: 10px;
        border-radius: 12px;
    }

</style>
# Need To Knows

<div class=important-boxed>
This means important. Pay close attention as it may be on the exam.
</div>

## Exam Breakdown {#exam-breakdown}
**Manage Azure identities and governance (15-20%)**
- ### [Manage AD objects](#manage-ad-objects)
- ### [Manage role-based access control (RBAC)](#manage-rbac)
- ### [Manage subscriptions and governance](#manage-subscriptions-and-governance)

**Implement and Manage Storage (10-15%)**
- ### [Manage Storage Accounts](#manage-storage-accounts)
- ### [Managed Data in Azure Storage](#manage-data-in-azure-storage)
- ### [Configure Azure files and Azure blob storage](#configure-azure-files-blob-storage)
---
### Manage AD Objects {#manage-ad-objects}
- [Create users and groups](#create-users-and-groups)
- [Manage user and group properties](#manage-user-and-group-properties)
- [Manage device settings](#manage-device-settings)
- [Perform bulk user updates](#perform-bulk-user-updates)
- [Manage guest accounts](#managing-guest-accounts)
- [Configure Azure AD join](#configure-azure-ad-join)
- [Configure self-service password reset](#configure-self-service-password-reset)

#### Create Users and Groups {#create-users-and-groups}

##### Cloud Identities
- Local Azure AD
    - Same tenant
- External Azure AD
    - Different tenant
##### Hybrid Identities
- Directory-synchronized
    - Think hybrid on-prem used with O365

##### Guest Identities
- Azure AD B2B collaboration
    - Identity management service for custom control of how customers sign up, authenticate, and manage their profiles
- External identities
    - Allows external

- MUST have User or Global administrator role

```PowerShell
# Create Azure AD User

Install-Module -Name AzureAD

Connect-AzureAD

$PasswordProfile = New-Object `
    -TypeName Microsoft.Open.AzureAD.Model.PasswordProfile

$PasswordProfile.Password = "P@ssw0rd8!"

$PasswordProfile.EnforceChangePasswordPolicy = $true

New-AzureADUser -DisplayName "Huy Vu" -PasswordProfile $PasswordProfile `
    -UserPrincipalName "huy@example.info" -AccountEnabled $true
```

##### Group
- Bulk add using CSV template in portal

##### Dynamic Group
- Rule based assignment
- Can be group-assigned roles and licenses
- CANNOT manually add users/devices

#### Manage User and Group Properties {#manage-user-and-group-properties}

- Modify user profile in the portal
- Modify group properties in the portal
- Modify using PowerShell
    - AzureAD module
- Must be User Administrator or Global Administrator role

#### Manage Device Settings {#manage-device-settings}

- Cloud device administrator
    - Add, Enable, disable, delete devices in AzureAD
    - Cannot modify properties
- Device administrator
    - Local machine administrator
    - Cannot modify object in Azure AD

#### Perform Bulk User Updates {#perform-bulk-user-updates}
- Bulk add to groups using CSV template in the portal
- Programmtically using PowerShell

#### Managing Guest Accounts {#managing-guest-accounts}
- Requires Azure AD Premium P2
- Guest Identities
    - Azure AD B2B collaboration
    - External identities
- Can be invited by:
    - Administrators
    - Users
- Roles required for guest review:
    - Global administrator
    - User administrator

#### Configure Azure AD Join {#configure-azure-ad-join}

| Azure AD Registered | Azure AD Joined |
| -- | -- |
| Personally onwed device | Organization owned device |
| MSA or local account sign-in | Azure AD sign-in |
| Windows 10<br>iOS<br>Android<br>macOS | Windows 10<br>Windows Server 2019 VMs in Azure |

#### Configure Self-Service Password Reset {#configure-self-service-password-reset}
 |  | Azure AD Free | Azure AD Premium P1 or P2 |
 | -- | -- | -- |
 | Cloud-only password change | :heavy_check_mark: | :heavy_check_mark: |
 | Cloud-only password reset | | :heavy_check_mark: |
 | Hybrid password change or reset with on-prem writeback | | :heavy_check_mark: |
 - Takeaway : licensing model

---
### Manage Role-Based Access Control {#manage-rbac}
- Skills Measured
    - Provide access to Azure resources by assigning roles
    - Interpret access assignments
    - Create a custom role

#### Security Principals
User
: individual with a profile in Azure AD
: can be assigned to users in other tenants

Group
: set of users created in Azure AD
: roles assigned to group impact all users

Security Principal
: security ID for applications or services
: used by applications or services to access specific resources

Managed Identity
: identity that is automatically managed by Azure AD
: Typically used for developing cloud apps to manage credentials for authenticating the app

#### Roles
Owner
: has full access to all resources and grant access

Contributor
: can create/manage all resources
: CANNOT grant access

Reader
: can view existing resources

User Access Administrator
: manaage user access

```PowerShell
New-AzRoleAssignment -SignInName janis.thomas@becausesecurity.com `
    -RoleDefinition "Virtual Machine Contributer" `
    -ResrouceGroupName ps-course-rg
```

#### Deny Assignments
- Blocks users from performaing specific actions even if a role assignement allows it
- Deny > Allow
- Created and managed in Azure to protect resources
- Can only be created using Azure Blue Prints or managed apps

#### Interpret Access Assignments
```PowerShell
Get-AzRoleAssignment -ResourceGroupName ps-course-rg
```
```bash
az role assignment list --resource-group ps-course-rg
```
```PowerShell
Get-AzRoleAssignment
Get-AzDenyAssignment
```
```bash
az role assignment list
```

#### Create a Custom Role
- Portal
    - Clone existing role
- ARM Template
- PowerShell
    - Modify existing role definition
    - Create new role using modified definition

##### Role Action Examples
| Operating String | Action |
| -- | -- |
| */read | Grants read access to all resource types of all resource providers |
| Microsoft.compute/* | Grants access to all operations for all resource types in the Microsoft.Compute resource provider | 
| microsoft.web/sites/restart/Action | Grants access to restart a web app |
> Be able to interpret the operation string for the exam

<br>

##### Create a custom role using powershell
```PowerShell
# Get Role Definition
$role = Get-AzRoleDefinition "Virtual Machine Contributor"
$role.Id = $null
$role.Name = "VM Reader"
$role.Description = "Can see VMs"

# Define new actions
$role.Actions.Clear()
$role.Actions.Add("Microsoft.Storage/*/read")
$role.Actions.Add("Microsoft.Network/*/read")
$role.Actions.Add("Microsoft.Compute/*/read")

$role.AssignableScopes.Clear()
$role.AssignableScopes.Add("/subscriptions/00000-1111-2222-aaaa-123456778")

New-AzRoleDefinition -Role $role
```
---

### Manage Subscriptions and Governance {#manage-subscriptions-and-governance}
- Skills measured
    - [Configure Azure policies](#azure-policy)
    - [Configure resource locks](#configure-resource-locks)
    - [Apply tags](#apply-tags)
    - [Create and manage resource groups](#create-and-manage-resource-groups)
    - [Manage subscriptions](#manage-subscriptions)
    - [Configure Cost Management](#cost-management)
    - [Configure management groups](#configure-management-groups)


#### Azure Policy {#azure-policy}
- Used to create, assign, and manage policies in Azure
- Enforce rules to ensure your resources remain compliant
- Focuses on resource properties for both new deployments and existing
- Does not apply remediations to resources that are not compliant

##### Policy Concepts
Policy Definition
: a rule in Azure Policy

Assignment
: an application of an initiative or a policy to a specific scope
: the action part of attaching to a particular scope

Initiative
: a collection of policy definitions
: are assigned to a specific scope
: can be assigned to different scopes across different subscriptions
```PowerShell
# Get reference to resource group that is the scope of the assignment
$rg = Get-AzResourceGroup -name $rgName

# Get reference to the built-in policy definition to assign
$definition = Get-AzPolicyDefinition | Where-Object {$_.Properties.DisplayName `
    -eq 'Audit VMs that do not use managed disks'}

# Create the policy assignement with the built-in definition against your resource group
New-AzPolicyAssignment -Name 'audit-vm-manageddisks' `
    -DisplayName 'Audit VMs without managed disks assignment' -Scope $rg.ResourceId `
    - PolicyDefinition $definition
```
#### Configure Resource Locks {#configure-resource-locks}
- Types:
    - Read-only
        - Can only READ the resource
    - Delete
        - Does NOT protect the data, files or database, from being deleted
- Inherited from parent scopes
    - For both existing and new resources
- Applies to all users and roles
```PowerShell
New-AzResourceLock -LockLevel CanNotDelete -LockName LockSite -Resourcename examplesite
```
```bash
az lock create --name LockGroup --lock-type CanNotDelete --resource-group exampleresourcegroup
```
#### Apply tags {#apply-tags}
- Used to organize resources and management heirarchy
- Consists of name/value pair
- MUST have write access to Microsoft.Resources/tags

#### Create and Manage Resource Groups {#create-and-manage-resource-groups}
- Can be moved from one resource group to another if it is supported
- Moving resources does not change the location/region where it was originally created
    - Only contains the metadata about the resources it contains
- Deleting a resource group deletes all resources within the deleted group
> IMPORTANT: ALL RESOURCES MUST HAVE A RESOURCE GROUP
```PowerShell
New-AzResourceGroup -Name example-rg -Location eastus2
```
```bash
az group create --name example-rg --location eastus2
```

#### Manage Subscriptions {#manage-subscriptions}
- Resources can be moved between subscriptions
- Subscriptions can be transferred between different tenants
- A single tenant can have multiple subscriptions

#### Cost Management {#cost-management}
- Analyze costs and trends using Cost Analysis
- Cost Alerts can be generated to alert when a threshold you define is met
- Apply Budgets to apply cost thresholds and limits to control you Azure spending
- Recommendations displays ways to control costs through identifying trends in your usage

#### Configure Management Groups {#configure-management-groups}
- Used to efficiently manage access, policies, and compliance
- Provides a level of scope over subscriptions
- Subscriptions with a group inherit policies applied to the group

### Manage Storage Accounts {#manage-storage-accounts}
- Objectives
    - [Configure network access to storage accounts](#configure-network-access-to-storage-accounts)
    - [Create and configure storage accounts](#create-and-configure-storage-accounts)
    - [Generate shared access signature (SAS)](#generate-shared-access-signature)
    - [Manage access keys](#manage-access-keys)
    - [Implement Azure storage replication](#implement-azure-storage-replication)
    - [Configure Azure AD Authentication for a storage account](#configure-azure-ad-auth-for-storage-account)

#### Configure Network Access to Storage Accounts {#configure-networok-access-to-storage-accounts}
- ##### Azure Storage Firewalls and Virtual Networks
    - Layered security model
    - Limit access by rules
        - IP addresses
        - IP ranges
        - Subnets in Azure vNets
    - Configured through Firewall and Virtual Networks blade
    - Service Endpoints in vNets
    - Requires Authorization

#### Create and Configure Storage Accounts {#create-and-configure-storage-accounts}
- ##### Create Storage Accounts
    - Contains all Azure storage objects
    - Accessed through unique namespace
    - Be familiar with creating storage account in Azure portal and Azure CLI
- ##### Types of Storage Accounts
    - General-purpose v2
        - This will be the go-to for most cases
    - General-purpose v1
    - BlockBlobStorage
    - FileStorage
    - BlobStorage
- ##### Access Tiers
    - Hot
        - Highest storage cost
        - Lower access cost
    - Cool
        - Lower storage cost
        - Higher access cost
        - 30 day minimum
    - Archive
        - Lowest storage cost
        - Highest access cost
        - 180 day minimum

<div class=important-boxed>

# **Important**
### Storage Account Supported Capabilities
|   | Data Objects | Performance Tiers | Access Tiers | Replication Options |
| - | - | - | - | - |
| General Purpose v2 | Blob, File, Table, Disk, Queue, & Data Lake Gen2 | Standard<br>Premium (Disk Only) | Hot, Cool. Archive | LRS, GRS, RA-GRS, ZRS, GZRS(preview), RA-GZRS(preview) |
| General Purpose v1 | Blob, File, Queue, Table, & Disk | Standard<br>Premium (Disk Only) | N/A | LRS, GRS, RA-GRS |
| BlockBlobStorage | Blob (block blobs and append blobs) | Premium | N/A | LRS, ZRS |
| FileStorage | File Only | Premium | N/A | LRS, ZRS |
| BlobStorage | Blob (block blobs and append blobs) | Standard | Hot, Cool, Archive | LRS, GRS, RA-GRS |
*Referenced from Microsoft at https://bit.ly/azstraccts*

</div>

### Manage Access Keys {#manage-access-keys}
- Access to entire storage account
    - Two keys created by default for storage account
    - Grants access to ENTIRE storage account, not just content
- Used in scenarios needing limited number of secrets
- Must be protected at all costs
- Consider Azure AD instead
- Use Azure Key Vault
    - Create managed key infrastructure
- Familiarize with retrieving and applying keys

### Generate Shared Access Signatures {#generate-shared-access-signature}
- Provide time-limited access to resources in a storage account
- Allows granular permissions
- can be applied at storage account or data object level
- Generated with multiple tools
    - Azure Portal
    - Azure Storage Explorer
- Familarize with generating and applying SAS keys

### Implement Azure Storage Replication {#implement-azure-storage-replication}
<div class=important-boxed>
<b>Know the 6 replication options<br>

| | | |
| -- | -- | -- |
| Local-Redundant Storage (LRS) | Zone-Redundant Storage(ZRS) | Geo-Redundant storage (GRS) |
| Geo-Zone-Redundant Storage (GZRS) | Read-Access Geo-Redundant Storage (RA-GRS) | Read-Access Geo-Zone-Redundant Storage (RA-GZRS) |

</b>

</div>
#### Replication Options in a Nutshell
- LRS and ZRS are SINGLE REGION replication only
    - No replication outside of Azure region
- ZRS provides replication across three Azure datacenters
- GRS and GZRS provide cross-region replication
- RA-GRS and RA-GZRS provide read-only access to replicated data
- Replication can be re-configured on Storage Account

<div class=important-boxed>

### Azure Storage Durability and Availability Scenarios

| Outage Scenario | LRS | ZRS | GRS/RA-GRS | GZRS/RA-GZRS |
| --------------- | --- | --- | ---------- | ------------ |
| Datacenter node becomes unavailable | Yes | Yes | Yes | Yes |
| Entire datacenter becomes unavailable | No | Yes | Yes | Yes |
| Primary region-wide outage | No | No | Yes | Yes |
| Read access in secondary region when primary is unavailable | No | No | Yes (with RA-GRS) | Yes (with RA-GZRS) |
*Referenced from Microsoft at https://bit.ly/2FeFav5*

</div>

### Configure Azure AD Authentication for a Storage Account {#configure-azure-ad-auth-for-storage-account}

- Supproted for Blob and Queue storage
- Uses RBAC
- Understand how permissions are applied
- Microsoft recommended approach
    - Use Azure AD whenever possible

<div class=important-boxed>

|      | Shared Key<br>(storage account Key) | Shared Access Signature (SAS) | Azure AD | Anonymous public read access |
| --- | --- | --- | --- | --- |
| Azure Blobs | Supported | Supported | Supported | Supported |
| Azure Files (SMB) | Supported | Not Supported | *Supported using Azure AD Domain Services ONLY | Not Supported |
| Azure Files (REST) | Supported | Supported | Not Supported | Not Supported |
| Azure Queues | Supported | Supported | Supported | Not Supported |
| Azure Tables | Supported | Supported | Not Supported | Not Supported |
*More information at https://bit.ly/2DHOGXa*

</div>

