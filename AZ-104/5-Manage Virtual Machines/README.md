[Creating a Virtual Machine](#1)
[Configure High Availability and Scalability](#2)
[Automate Deployment and Configuration of Virtual Machines](#3)


# Creating a Virtual Machine {#1}

- Still requires maintenance
- No need to worry about hardware
- Design considerations
    - Availability
    - Scalability
- Uses VHDs
- Extensions available for Azure VMs
    - Custom Script extensions
    - Diagnostics/Monitoring

## Resources Used by VMs
| Resource | Required | Description |
| :------- | :------: | :---------- |
| Resource Group | Yes | Must be contained in a resource Group |
| Storage Account | Yes | Needs storage account |
| Virtual network | Yes | Must be a member of a network |
| Public IP | No | Can have a PIP assigned to access remotely |
| Network Interface | Yes | The VM needs a NIC to communicate |
| Data Disks | No | Can include data disks to expand storage |
*Reference: https://bit.ly/36tgf2u*

###### Powershell
```PowerShell
# Creating a VM
New-AzVM -ResourceGroupname 'ps-course-rg' -Name 'windows-1'`
    -Location 'SouthCentralUS' -VirtualNetworkName 'main-vnet'`
    -SubnetName 'backend' -SecurityGroupName 'myNetworkSecurityGroup'`
    -PublicIpAddressName 'myPublicIpAdress' -OpenPorts 80,3389

# Because we are not defining an image or OS, Azure will default to Windows Server 2016
```
###### Azure CLI
```Bash
az vm create --resource-Group ps-course-rg --name windows1 \
    --image win2016datacenter --admin-username azureuser
```

## Moving a VM
- All v-net resources must be moved when moving to a different subscription. Does not apply to a resource group.
- VM scale sets with standard Load balancers and standard PIP cannot be moved
- VMs integrated with key vault for disk encryption cannot be moved. Encryption must be disabled before VM can be moved.
- VMs configured with Azure backup must have restore points deleted prior to moving

###### Powershell
```PowerShell
Move-AzResource -DestinationResourceGroupname 'ps-course-rg' `
    -ResourceId <myResourceId1,myResourceId2,myResourceId3>
# This cmdlet supports moving multiple resources such as network resources being moved along with VM

Move-AzResource -DesintationSubscriptionId "8bc4fbf0-some-more-char-226j44r4bg93" `
    -DestinationResourceGroupName 'ps-course-rg' `
    -ResourceId <myResourceId1,myResourceId2,myResourceId3>
```

## Manage VM Sizes
- Select the VM
- Select Size blade under Settings
- Select size and select resize
- The VM will reboot to the resized VM
*Best practices are to shut down VM before resizing*

## Redeploy VMs
<dl>
    <dt>What redeploy does</dt>
    <dd>shuts down VM, moves to new node, and powers back up</dd>
</dl>

When to use:
- Cannot connect via RDP or SSH
- Difficulty troubleshooting application access on Azure Vm
###### PowerShell
```PowerShell
Set-AzVM -Redeploy -ResourceGroupname 'ps-course-rg' -Name 'linux-1'
```
###### Azure CLI
```Bash
az vm redeploy --resource-Group ps-course-rg --name linux-1
```

# Configuring VMs
[Configure Networking](#configure-networking)
[Add Data Disks](#add-data-disks)
[Configure Azure Disk Encryption](#configure-azure-disk-encryption)

## Configure Networking {#configure-networking}
<dl>
    <dt>Required network resources</dt>
    <dd>Network Interface</dd>
    <dd>Virtual Network and Subnets</dd>
    <dd>IP Address</dd>
</dl>

*Note: deallocating VM releases dynamic public IP to be reassigned*

- When creating an Azure VM, you MUST create a virtual network or use an existing VNet
- No security boundary between subnets by default
- To add a NIC to an existing VM, it must first be deallocated
- NIC can only be assigned to a virtual network that exists in the same location as the NIC

```Bash
# Azure CLI Basic Public IP
az network public-ip create -g ps-course-rg -n MyIp
```

## Attaching Data Disks
- Can add new or existing data disks
- Done through portal and CLI
- Adding managed disks allows you to choose from source types of BLOB or snapshots
- Disks can be added on the fly
    - Changes to OS disk requires stopping VM

## Azure Disk Encryption {#configure-azure-disk-encryption}
- Uses Bitlocker for Windows
- Uses DMCrypt for Linux
- Full disk encryption of OS and data disk
- Integrated with Azure Key Vault
- VMs must be able to connect to either Azure AD or the KeyVault endpoint
- Azure KeyVault access policy must be enabled for Azure Disk Encryption

###### PowerShell - Enabling disk encryption 
```PowerShell
New-AzKeyVault -Name 'demokv' -ResourceGroupname 'ps-course-rg' `
    -Location 'southcentralus' -EnabledForDiskEncryption

$KeyVault = Get-AzKeyVault -VaultName 'demokv' -ResourceGroupname 'ps-course-rg'

Set-AzVMDiskEncryptionExtension -ResourceGroupname 'ps-group-rg' -VMName 'linux-1' `
    -DiskEncryptionVaultUrl $KeyVault.VaultUri `
    -DiskEncryptionVaultId $KeyVault.ResourceId
```

# Configure High Availability and Scalability {#2}

## High Availability Constructs
- [Availability Zones](#availability-zones)
- [Fault Domains](#fault-domains)
- [Update Domains](#update-domains)
- [Availability Sets](#availability-sets)
- [Scale Sets](#scale-sets)

### Availability Zones {#availability-zones}
- One or more datacenters deployed across Azure regions
    - Good for application deployments
    - 3 zones per region
- Standard SKU load balancers are availability zone aware
- Standard SKU PIPs are required

![Availability Zones](https://docs.microsoft.com/en-us/azure/availability-zones/media/az-overview/az-graphic-two.png)

##### SLA agreement

> For all Virtual Machines that have two or more instances deployed across two or more Availability Zones in the same Azure region, we guarantee you will have Virtual Machine Connectivity to at least one instance at least 99.99% of the time.

*Reference: https://azure.microsoft.com/en-us/support/legal/sla/virtual-machines/v1_9/*

### Fault Domains {#fault-domains}
-  Local group of hardware in an Azure datacenter
    - Essentially a rack of servers in an Azure datacenter
- VMs in the same fault domain share common power source and physical network switch

### Update Domains {#update-domains}
- Individual servers
- Protects against normal maintenance updates
    - Applying updates to hardware
- VMs created in the same update domain will be restarted together during planned maintenance
- Only one update domain restarted at a time

### Availability Sets {#availability-sets}
- Groups VMs to distribute across a single datacenter
    - Minimize disruptions caused by maintenance or outages
    - Good for VMs
- Cannot add a VM to availability set post deployment
    - MUST be configured at creation or time of deployment

![Availability Sets](https://docs.microsoft.com/en-us/azure/virtual-machines/media/virtual-machines-common-manage-availability/ud-fd-configuration.png)

##### SLA agreement
> For all Virtual Machines that have two or more instances deployed in the same Availability Set or in the same Dedicated Host Group, we guarantee you will have Virtual Machine Connectivity to at least one instance at least 99.95% of the time.

*Reference: https://azure.microsoft.com/en-us/support/legal/sla/virtual-machines/v1_9/*

### Scale Sets {#scale-sets}
- Group of load balanced VMs
- Can scale automatically based on demand or schedule
- 2 or more VMs recommended
- No additional cost other than extra instances
- Can be deployed across multiple update/fault domains

# Automate Deployment and Configuration of Virtual machines {#3}

## ARM Templates
- JSON format
- Contains the resources to create in Azure
- Submit template to the Azure Resource Manager (ARM)
- Tracked deployments

##### ARM template format
```JSON
{
    "$schema": "https://schema.management.azure.com/schemas/2019-04-20/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {},
    "functions": {},
    "variables": {},
    "resrouces": [],
    "outputs": {}
}
```
Required: **schema, contentVersion, resources**

<dl>
    <dt>Schema</dt>
    <dd>defines the location of the JSON file that describes the version of the language
    <dt>contentVersion</dt>
    <dd>Defines the version of the specific iteration of the file</dd>
    <dt>Resources</dt>
    <dd>Define the resources that will be deployed or updated</dd>
</dl>

Arm templates can used to update existing resources, as well as creating new resources.

##### Example Template
```JSON
{
    "schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "networkSecurityGroups_web_nsg_name": {
            "defaultValue": "web-nsg",
            "type": "String"
        }
    },
    "variables": {},
    "resources": [
        {
            "type": "Microsoft.Network/networkSecurityGroups",
            "apiVersion": "2020-05-01",
            "name": "[parameters('networkSecurityGroups_web_nsg_name')]",
            "location": "centralus",
            "tags": {
                "environment": "production"
            },
            "properties": {
                "securityRules": [
                    {
                        "name": "Port_443",
                        "properties": {
                            "protocol": "*",
                            "sourcePortRange": "*",
                            "destinationPortRange": "443",
                            "sourceAddressPrefix": "*",
                            "destinationAddressPrefix": "*",
                            "access": "Allow",
                            "priority": 100,
                            "direction": "Inbound",
                            "sourcePortRanges": [],
                            "destinationPortRanges": [],
                            "sourceAddressPrefixes": [],
                            "destinationAddressPrefixes": []
                        }
                    }
                ]
            }
        },
        {
            "type": "Microsoft.Network/networkSecurityGroups/securityRules",
            "apiVersion": "2020-05-01",
            "name": "[concat(parameters('networkSecurityGroups_web_nsg_name'), '/Port_443')]",
            "dependsOn": [
                "[resourceId('Microsoft.Network/networkSecurityGroups', parameters('networkSecurityGroups_web_nsg_name'))]"
            ],
            "properties": {
                "protocol": "*",
                "sourcePortRange": "*",
                "destinationPortRange": "443",
                "sourceAddressPrefix": "*",
                "destinationAddressPrefix": "*",
                "access": "Allow",
                "priority": 100,
                "direction": "Inbound",
                "sourcePortRanges": [],
                "destinationPortRanges": [],
                "sourceAddressPrefixes": [],
                "destinationAddressPrefixes": []
            }
        }
    ]

}
```

## Deploy from Template
