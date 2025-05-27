# AZ VM's, Virtual Networks, Subnets
[<img src="../images/back.png">](../README.md)

## Traditional non Bastion setup

<img title="Subnet segregation and Multiple Network Interfaces" alt="Alt text" src="images/multiple-nic.png">

## Bastion Host vs Azure Bastion Service

<img title="Bastion Host vs Azure Bastion Service" alt="Alt text" src="images/bastion.png">

### Azure Bastion Service
Azure Bastion eliminates the need to set up VPNs, jump servers, or additional security layers by providing a managed, browser-based RDP and SSH connection over SSL. It enables administrators to manage their VMs without needing a public IP address, making the network more secure and less vulnerable to threats. This approach simplifies secure access while maintaining a robust security posture.

### Bastion Host
Setting up a Bastion Host follows a traditional approach where a dedicated virtual machine is deployed in an isolated subnet, separate from the main workload VMs. This Bastion Host has a public IP exposed to the user, enabling secure access to this isolated VM. From the Bastion Host, the user can then connect to the internal workload VMs, ensuring secure connectivity. Once the Bastion Host is configured, the public IP of the main resources is removed, minimizing exposure to the public internet while maintaining secure, internal access to the VMs.


[<img src="../images/back.png">](../README.md)