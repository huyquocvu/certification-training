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
>Disclaimer: This README contains styling that does not work with Github-flavored Markdown.
# Need To Knows

<div class=important-boxed>
This means important. Pay close attention as it may be on the exam.
</div>

## Exam Breakdown {#exam-breakdown}
**Manage Azure identities and governance (15-20%)**
- [Manage AD objects](#manage-ad-objects)
- [Manage role-based access control (RBAC)](#manage-rbac)
- [Manage subscriptions and governance](#manage-subscriptions-and-governance)

**Implement and Manage Storage (10-15%)**
- [Manage Storage Accounts](#manage-storage-accounts)
- [Managed Data in Azure Storage](#manage-data-in-azure-storage)
- [Configure Azure files and Azure blob storage](#configure-azure-files-blob-storage)

**Deploy and manage Azure compute resources (25-30%)**
- [Create and configure VMs](#create-configure-VMs)
- [Configure VMs for high availability and scalability](#configure-vms-for-high-availability)
- [Automate deployment and configuration of VMs](#automate-deployment-configuration-vms)
- [Create and configure containers](#create-configure-containers)
- [Create and configure Web Apps](#create-configure-web-apps)


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

### Create and Configure VMs {#create-configure-VMs}
- [Configure Azure Disk Encryption](#configure-azure-disk-encryption)
- [Move VMs from one resource group to another](#move-vms)
- [Manage VM sizes](#manage-vm-sizes)
- [Add data disks](#add-data-disks)
- [Configure networking](#configure-networking)
- [Redeploy VMs](#redeploy-vms)

#### Configure Azure Disk Encryption {#configure-azure-disk-encryption}
- Full disk encryption of the OS and data disk
- Azure disk encryption is integrated with Azure Key Vault
- VMs must be able to connect to either Azure AD or the KeyVault endpoint
#### Move VMs from One Resource Group to Another {#move-vms}
- Moving a VM to another subscription requires moving all dependent items
- VM scale sets with standard load balancers/PIPs cannot be moved
- VMs integrated with key vault for disk encryption cannot be moved
```PowerShell
Move-AzResource -DestinationResourceGroupName 'ps-course-rg' `
    -ResourceId <myResourceId,myResourceId,myResourceId>

Move-AzResource -DestinationSubscriptionId "some-subscription-id" `
    -DestinationResourceGroupName 'ps-course-rg' `
    -ResourceId <myResourceId,myResourceId,myResourceId>

# Moving multiple resources can be done via comma separated list
```
>Be familiar with these PowerShell commands
#### Manage VM Sizes {#manage-vm-sizes}
VM will reboot after being resized.
That is all.
#### Add Data Disks {#add-data-disks}
- Can add a new or existing data disk
- Adding managed disks allows you to choose from source types of BLOB or snapshots
#### Configure Networking {#configure-networking}
- When creating an Azure VM, you must create a virtual network or use an existing vNet
- There is no security boundary between subnets by default
- To add a NIC to an existing VM, it must first be deallocated
    - Can use PowerShell with '-force' parameter, then add NIC
- A deallocated VM releases dynamically assigned public IPs
- A NIC can only be assigned to a virtual network that exists in the same location as the NIC
#### Redeploy VMs {#redeploy-vms}
When to use redploy option:
- Cannot connect via RDP or SSH
- Redeploy shuts down the VM and moves to new node and powers back up

```powershell
# PowerShell
Set-AzVM -Redeploy -ResourceGroupName 'ps-course-rg' -Name "linux-1"

# Azure CLI
az vm redeploy --resource-group ps-course-rg --name linux-1
```

### Configure VMs for High Availability and Scalability {#configure-vms-for-high-availability}

- Configure VMs for High Availability
- Deploy and configure scale sets

#### Configure VMs for High Availability
Availability Zones
- Distribute VMs across *Azure regions*
    - 3 zones per region
- Standard SKU load balancers are availability zone aware
- Standard SKU PIPs are required

Fault Domains
- Logical group of hardware in an Azure datacenter
- VMs in the same fault domain share common power source and physical network switch

Update Domains
- Protect against normal maintenance updates
- VMs created in the same update domain will eb restarted together during planned maintenance
- Only one update domain restarted at a time

Availability Sets
- Group VMs to distribute across a single datacenter
- 5 update domains assigned by default
    - Can provide up to 20
- Cannot add a VM to availability set post-deployment
    - Must be done at creation

Scale Sets
- Group of load balanced virtual machines
- Can scale automatically based on demand or schedule
- 2 or more VMs recommended
- Can be deployed across multiple update/fault domains

### Automate Deployment and Configuration of VMs {#automate-deployment-configuration-vms}
Modify ARM template
- JSON format
- Used to create or modify resources in Azure
- Submit template to the Azure Resource Manager (ARM)
- Can modify existing template in portal
    - Choose Export template under Automation
    - Select Deploy to edit template
    - Make changes and save

Deploy from template
- Generate template in the protal
- Download the template
- Edit and deploy modified template

Save a deployment as an ARM template
- Locate resource group in the portal
- Choose Export template
- Download template

Automate configuration management by using custom script extension
- Scripts can be located anywhere as long as VM can access it
- Scripts can be deployed with ARM template
- Script will only run once

Configure VHD template
- Sysprep managed impage with supprot up to 20 simultaneous deployments
- Capture image, provide image name
- Choose to have VM deleted after capture
- Provide VM name to confirm the process

### Create and Configure Containers {#create-configure-containers}
Create and Configure Azure Containers
- Restart Policies
    - Always
    - On failure
    - Never
```PowerShell
# Create a resource group
az group create --name ps-course-rg --location centralus

# Create and deploy container
az container create --resource-group ps-course-rg --name mycontainer \
    --image mcr.microsoft.com/azuredocs/aci-helloworld --dns-name-label az104-demo \
    --ports 80 --restart-policy Always
```

Create and Configure Azure Kubernetes Service
- AKS cluster must use VM scale sets for the nodes for autoscaling and multiple node pools
- All node pools must reside in the same virtual network
- **AKS cluster must use the *Standard SKU* load balancer to use multiple node pools**
```powershell
# Create a basic single-node AKS cluster
az aks create \
    --resource-group ps-course-rg \
    --name PSAKSCluster \
    --vm-set-type VirtualMachineScaleSets \
    --node-count 2 \
    --generate-ssh-keys \
    --load-balancer-sku standard
```

### Create and Configure Web Apps {#create-configure-web-apps}
Create and configure App Service Plans

<div class=important-boxed>

| Features | Free (F)/Shared (D) | Standard | Premium v2 | Premium v3 |
| -------- | ----------- | -------- | ---------- | ---------- |
| Custom Domain | Shared D, B | Yes | Yes | Yes |
| Scale | B manual (3) | Auto 10 | Auto 20 | Auto 30 |
| Staging slots | | 5 | 20 | 20 |
| Daily Backups | | 10 | 50 | 50 |
| Traffic Manager | | Yes | Yes | Yes |

</div>

```powershell
# Create resource group
az group create --name ps-app-rg --location centralus

# Create app service plan
az appservice plan create --name psasp --resource-group ps-app-rg --sku F1 --is-linux

# Create web app
az webapp create --name dotnetapp --resource-group ps-app-rg --plan psasp
```

Create and configure App Service
- Web app and App Service Plan needs to be in the same region
- Cannot mix Windows and Linux apps in the same App Service Plan
- .NET Core is supported on both Windows and Linux
- Autoscaling is determined by rules based on threshold metrics defined

Good to know:
```powershell
# Creating a VM using PowerShell
# Note the patterns
# Creating a new resource will start with the "New" verb
# Retrieving information about a resource will start with the "Get" verb

New-AzResourceGroup -Name 'ps-course-rg' -Location 'CentralUS'
New-AzVM -ResourceGroupName 'ps-course-rg' -Name 'windows-1'`
    -Location 'CentralUS' -VirtualNetworkName 'main-vnet'`
    -SubnetName 'backend' -SecurityGroupName 'myNetworkSecurityGroup'`
    -PublicIpAddressName 'myPublicIpAddress' -OpenPorts 80,3389

# Using Azure CLI
az group create --name ps-course-rg --location centralus
az vm create --resource-Group ps-course-rg --name windows-1 \
    --image win2016datacenter --admin-username azureuser
```
Strategy:
- Know what product SKUs are required for services
    - Availability zones
    - App service plans
- Be familiar with implementations in portal with code