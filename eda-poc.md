### Introduction
The Azure cloud is an excellent environment to run large scale engineering workloads such as HPC and EDA. Azure implements several features that facilitate its usage for HPC workloads. While EDA workloads have a slightly different profile than conventional HPC workloads, some of the capabilities Azure implements for HPC can be leveraged by EDA. Those capabilities, combined with speciality solutions Azure develops with semiconductor workloads in mind, make Azure ideal for running chip design. The following document provides step by step guidance for how to migrate an EDA workload to the Azure cloud. This is meant to provide one example of a successful implementation, but is not meant to be exhuastive or illustrative of all possibilities. 

### Prerequisites
Have a new subscription with Owner/Contributor privileges



#### Getting Started
Head over to Azure Portal https://portal.azure.com

Click on Azure Cloud Shell Icon. 
We will be creating a bunch of Azure resources to standup our HPC/EDA environment

Set the followinig variables

    RESOURCE_GROUP="rg-eda"

    REGION="westus3"

    VNET_NAME="MyVNet"


#### Resource group
    az group create -l $REGION -n $RESOURCE_GROUP

#### Virtual Network
    az network vnet create -l $REGION -g $RESOURCE_GROUP -n $VNET_NAME --address-prefix 10.0.0.0/16 --subnet-name MySubnet --subnet-prefixes 10.0.0.0/24

#### Base Image
Create a new virtual machine from Azure Marketplace (CentOS, Rocky Linux, AlmaLinux, etc..) 

Install your custom tools as well as the library dependencies required by your tool vendor. Follow the steps mentioned in this document to generalize your VM

https://learn.microsoft.com/en-us/azure/virtual-machines/generalize

#### Compute Image Gallery
We will use Azure's Compute Image Gallery(CIG) and store and version the managed imanges. It'll also be helpful to distribute these images across the azure regions.
https://learn.microsoft.com/en-us/cli/azure/sig?view=azure-cli-latest#az-sig-create()

    # Create an Image Gallery
    az sig create -g $RESOURCE_GROUP --gallery-name MyGallery -l $REGION

    # Create Image Definition
    az sig image-definition create -g $RESOURCE_GROUP --gallery-name MyGallery --gallery-image-definition centosimagedef --publisher KA --offer KA --sku KA --hyper-v-generation V2 --os-type linux --os-state Generalized

    # Create Image Version
    az sig image-version create -g $RESOURCE_GROUP --gallery-name MyGallery --gallery-image-definition centosimagedef --gallery-image-version 1.0.0 --managed-image {resource-id of your generalized image} --target-regions $REGION=1=standard_lrs --target-region-encryption MyDiskEncryptionSet --no-wait


#### Azure NetApp Files
EDA workloads rely on high performance shared file storage. NFS is the industry standard for EDA and the Azure NetApp Files (ANF) provides the performance required to run EDA workloads on the cloud at scale. ANF is widely used as the underlying shared file-storage service in various scenarios. These include migration (lift and shift) of POSIX-compliant Linux and Windows applications, SAP HANA, databases, high-performance compute (HPC) infrastructure and apps, and enterprise web applications. 

Make sure ANF is provisioned in the same the resource group, region and VNet. We need a dedicated subnet before provisioning ANF. 

    az network vnet subnet create -g $RESOURCE_GROUP --vnet-name MyVnet -n anfsubnet --address-prefixes 10.0.1.0/26

Follow the below document to provision ANF. 

https://learn.microsoft.com/en-us/azure/azure-netapp-files/azure-netapp-files-quickstart-set-up-account-create-volumes?tabs=azure-portal


#### User Managed Identity
A common challenge for developers is the management of secrets, credentials, certificates, and keys used to secure communication between services. Managed identities eliminate the need for developers to manage these credentials.

Following this page to create a new "User Managed Identity"

https://learn.microsoft.com/en-us/azure/active-directory/managed-identities-azure-resources/overview

    az identity create -g $RESOURCE_GROUP -n myUserAssignedIdentity
    
https://learn.microsoft.com/en-us/azure/active-directory/managed-identities-azure-resources/qs-configure-cli-windows-vm


#### Disk Encryption Set

https://learn.microsoft.com/en-us/azure/virtual-machines/disks-enable-customer-managed-keys-portal

    az disk-encryption-set create --resource-group $RESOURCE_GROUP --name MyDiskEncryptionSet --key-url {Vault URI} --source-vault MyVault

#### Azure Key Vault
Follow this document to create an Azure Key Vault and keys

https://learn.microsoft.com/en-us/azure/key-vault/general/quick-create-cli

Give the Azure RBAC  (Key Vault Secrets User and Key Vault Reader) roles on the Azure KV by selecting managed identity of Disk Encryption Set

Follow this doc for more info
https://learn.microsoft.com/en-us/azure/key-vault/general/rbac-guide?tabs=azure-cli


#### Storage account. 

Depending on your organization needs you can make the Azure KV and Storage Account private link enabled.

Follow these steps below if you need to create a Storage Account with Customer Managed Keys.

https://learn.microsoft.com/en-us/azure/storage/common/customer-managed-keys-configure-new-account?tabs=azure-portal



#### Azure Bastion
Azure Bastion is a fully managed service that lets you connect to  virtual machines using your browser and the Azure portal, or via the native SSH or RDP client already installed on your local computer. It needs a dedicated subnet and should be named as AzureBastionSubnet

    az network vnet subnet create -g $RESOURCE_GROUP --vnet-name MyVnet -n AzureBastionSubnet --address-prefixes 10.0.2.0/27

Follow the steps mentioned in this document to create an Azure Bastion instance
    
https://learn.microsoft.com/en-us/azure/bastion/tutorial-create-host-portal


#### Azure CycleCloud
EDA workloads often rely on a job scheduler for manage the scale-out of engineering workloads. Job schedulers, such as LSF, GridEngine, etc. are often used in EDA. CycleCloud uses the scheduler queues to understand when to provision and deprovision VMs to process the jobs in the queues with only the resources required. Provision a CycleCloud from Azure Marketplace. Choose the latest version.
You can create a VM without a public IP so that this VM is not exposed to public internet. Alternatively you can peer this VNet to an existing VNet that's connected to your VPN or Express Route. This allows you to access this VM from your on-premise network.

After the VM is successfully provisioned, access the VM on it's private IP.

Follow the steps below to provision and configure a CycleCloud VM

https://learn.microsoft.com/en-us/azure/cyclecloud/qs-install-marketplace?view=cyclecloud-8


#### License Server (Cadence, Ansys)
Similar to the above step, provision another Linux VM (CentOS, Ubuntu etc.) and install the relevant software to managed your licenses

