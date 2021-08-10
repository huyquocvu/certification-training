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
