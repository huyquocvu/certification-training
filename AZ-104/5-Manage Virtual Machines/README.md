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


