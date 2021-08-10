# Managing Subscriptions

### Types of subscriptions
- Free Trial
- Support Plan
- MSDN
- Visual Studio
- Pay-as-you-go
- Enterprise Agreement (EA)
---
- You can move resources between subscriptions
- You can transfer subscriptions between different tenants
- A single tenant can have multiple subscriptions
---
## Cost Management
[What is Azure Cost Management + Billing?](https://docs.microsoft.com/en-us/azure/cost-management-billing/cost-management-billing-overview)

Cost Analysis
: Tool for analyzing costs and trends

Cost Alerts
: Alert generated when a cost threshold is met

Budgets
: Applies cost thresholds and limits 

Recommendations
: Displays ways to control costs through identitying trends in usage

---

## Tagging

- Used to organize resources and management hierarchy
- Each tag consists of name/value pairs
- Must have write access to Microsoft.Resources/tags
- Types are not inheritable

<br>

Use PowerShell to recursively apply tags
```PowerShell
# Get tagged resources
Get-AzTag -Detailed

# Get resouces by tag name
(Get-AzResource -Tagname $tagName).name

# Get resources by tag value
(Get-AzResource -TagValue $tagValue).name

# Add an existing tag and non-existing tag to a resource with a tag already
$tags = @{'project' = 'ux'; 'location' = 'Dublin'}
$rg = Get-AzResourceGroup -Name 'demo-rn'
$rg.ResourceId
New-AzTag -ResourceId $rg.ResourceId -Tag $tags # This will overwrite existing tags

# Add tags to resources within a resource group
Get-AzResource -ResourceGroupName 'demo-rg' | Set-AzResource -Tag @{'environment' = 'staging'}
```
---
# Managing Governance

[What are Azure management groups?](https://docs.microsoft.com/en-us/azure/governance/management-groups/overview)

[Create a management group with Azure Powershell](https://docs.microsoft.com/en-us/azure/governance/management-groups/create-management-group-powershell)

### Azure Management Groups
- Used to manage access, policies, and compliance
- Provides a level of scope over subscriptions
- Subscriptions within a group inherit policies applied to the group

### Root Management Groups
- Each directory has a single top-level group called the "Root"
- Allows for global policies and RBAC assignments at the directory level
- AD Global admin needs to elevate to User Access Administrator role
- Root management group CANNOT be moved or deleted

## Azure Policy
- Used to create, assign, and manage policies in Azure
- Enforce rules to ensure your resources remain compliant
- Focuses on resource properties for both new deployments and existing
- DOES NOT apply remediations to resources that are not compliant but suggests remediations

### Policy concepts

Policy Definition
: a rule

Assignment
: an application of an initiative or a policy to a specific scope

Initiative
: a collection of policy definitions

### Azure Policy Creation - Powershell
```PowerShell
# Get a reference to the resource group that is the scope of the assignment
$rg = Get-AzResourceGroup -Name $resourceGroupName

# Get a reference to the bult-in policy definition to assign
$definition = Get-AzPolicyDefinition | Where-Object{$_.Properties.DisplayName -eq 'Audit VMs that do not use managed disks'}

# Create the policy assignment with the built-ion definition against your resource group
New-AzPolicyAssignment -Name 'audit-vm-manageddisks' -DisplayName 'Audit VMs without managed disks Assignment' -Scope $rg.ResourceId -PolicyDefinition $definition
``` 
> ##### Exam tip
> - Get familiar with the sequencing of the commands
> - Most likely involves starting with creating or getting a resource group

---

## Resource Locks

- Each resource can have a lock applied
- Lock Types
    - Read-only
    - Delete
- Can apply to all resource and resource groups
- Can be inherited from parent scopes
    - For both existing and new resources
- Applies to all users and roles

###### PowerShell
```PowerShell
New-AzResourceLock -LockLevel CanNotDelete -LockName LockSite -ResourceName examplesite
```
###### Azure CLI
```bash
az lock create --name LockGroup --lock-type CanNotDelete --resource-group exampleresourcegroup
```
---
## Resource Groups
- Containers that hold related Azure resources
- Resources can be moved from one resource group to another if it is supported
- Moving resources does not change the location/region where it was originally created
    - It stores the metadata about the resource it contains
- Deleting a resource group deletes ALL resources within the deleted group

###### PowerShell
```PowerShell
New-AzResourceGroup -name example-rg -location eastus2
```

###### Azure CLI
```bash
az group create --name example-rg --location eastus2
```