# Creating a Virtual Machine

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
