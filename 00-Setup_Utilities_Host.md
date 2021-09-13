# Objectives
An Azure VM is required as an utilities host serving the following functions:
- Jump host to access the AKS environment
- Docker host to run the Conjur master container
- MySQL database server

## Install Jump Host.
1.0. Login to your Azure Portal (https://portal.azure.com)

2.0. Setup Azure Environment

2.1. Create Azure Resource Group

Provide a name for your new resource group
![image](https://github.com/rajnishgargcloudrepository/conjur-aks/blob/main/images/00-Create-a-resource-group.png)

Add tags as desired for your new resource group

2.2. Create Azure Virtual Network

Provide a name for your new virtual network and associate it with the resource group created in 2.1
![image](https://github.com/rajnishgargcloudrepository/conjur-aks/blob/main/images/00-Create-virtual-network-Basics.png)

Configure IP address space and subnet for the virtual network
![image](https://github.com/rajnishgargcloudrepository/conjur-aks/blob/main/images/00-Create-virtual-network-IP-Addresses.png)

Leave security as default or customize as desired.

Add tags as desired for your new resource group

3.0. Setup Azure VM

3.1. Create a virtual Machine - Basics
- Resource group: Select the resource group created in 2.1
- Image: Ubuntu Server 20.04 LTS - Gen2 (You may choose other Linux distributions as long as kubectl, Docker and MySQL are supported on it. However, note that the steps in this guide are for Ubuntu.)
- Size: Standard_D2s_v3 (This size is meant for lab or development test, if using for production, do refering to CyberArk sizing guide.)
**WARNING** The Azure VM will incur costs in your Azure subscription.
- Administrator account: You can customize to use password or SSH key authentication, choose a username, and choose to generate new or use an existing key pair as desired.
![image](https://github.com/rajnishgargcloudrepository/conjur-aks/blob/main/images/00-Create-a-virtual-machine-Basics.png)

3.2. Create a virtual Machine - Disks
- Disk options: Select the disk and encryption type as desired
- Data disks: We do not need additional data disks
- Advanced: Use managed disks to enable data persistence
![image](https://github.com/rajnishgargcloudrepository/conjur-aks/blob/main/images/00-Create-a-virtual-machine-Disks.png)

3.3. Create a virtual Machine - Networking
- Virtual network: Select the VNet created in 2.2
- Subnet: Select the subnet created in 2.2
- Public IP: Allow the wizard to create a new public IP, or use your existing public IP, as desired
- Public inbound ports: For lab testing purposes of this guide, we will allow access to our VM on SSH. Do follow your organization's access policy accordingly in providing access to cloud instances.
![image](https://github.com/rajnishgargcloudrepository/conjur-aks/blob/main/images/00-Create-a-virtual-machine-Networking.png)

3.4. Create a virtual Machine - Create

- Leave Management and Advanced as default or customize as desired.
- Add tags as desired for your new VM.
- Review the configurations and create the Azure VM.

3.5. Verify access to the Azure VM

![image](https://github.com/rajnishgargcloudrepository/conjur-aks/blob/main/images/00-Create-a-virtual-machine-PuTTY.png)
