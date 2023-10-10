##### Introduction
The following document is written to onboard EDA workloads in your Azure subscription

### Prerequisites
Have a new subscription with Owner/Contributor privileges



#### Getting Started
Head over to Azure Portal https://portal.azure.com

Click on Azure Cloud Shell Icon. 
We will be creating a bunch of Azure following resources to standup our HPC/EDA environment

Set the followinig variables

    RESOURCE_GROUP=xx

    REGION=

    VNET_NAME=


#### Resource group
    az group create -l $REGION -n $RESOURCE_GROUP

#### Virtual Network
    az network vnet create -l $REGION -g $RESOURCE_GROUP -n $VNET_NAME --address-prefix 10.0.0.0/16 --subnet-name MySubnet --subnet-prefixes 10.0.0.0/24

#### Base Image
Create a new virtual machine from Azure Marketplace (CentOS, Ubuntu etc..) 
Install your custom tools/software. Follow the steps mentioned in this document to generalize your VM
https://learn.microsoft.com/en-us/azure/virtual-machines/generalize

#### Azure Key Vault
Follow this document to create an Azure Key Vault and keys

https://learn.microsoft.com/en-us/azure/key-vault/general/quick-create-cli

#### Storage account. 

Depending on your organization needs you can make the Azure KV and Storage Account private link enabled.
Follow these steps below if you need create a Storage Account with Customer Managed Keys.
https://learn.microsoft.com/en-us/azure/storage/common/customer-managed-keys-configure-new-account?tabs=azure-portal

#### Azure NetApp Files
Azure Netapp Files (ANF) is recommended to run HPC workloads. We need a dedicated subnet to provision ANF. 

Make sure ANF is provisioned in the same the resource group, region and VNet.

Follow the below document to provision ANF. 

https://learn.microsoft.com/en-us/azure/azure-netapp-files/azure-netapp-files-quickstart-set-up-account-create-volumes?tabs=azure-portal

#### Disk Encryption Set
    az disk-encryption-set create --resource-group $RESOURCE_GROUP --name MyDiskEncryptionSet --key-url {Vault URI} --source-vault MyVault

#### Compute Image Gallery
https://learn.microsoft.com/en-us/cli/azure/sig?view=azure-cli-latest#az-sig-create()

az sig create -g $RESOURCE_GROUP --gallery-name MyGallery -l $REGION

    ##### Create Image Definition
    az sig image-definition create -g $RESOURCE_GROUP --gallery-name MyGallery --gallery-image-definition centosimagedef --publisher KA --offer KA --sku KA --hyper-v-generation V2 --os-type linux --os-state Generalized

    ##### Create Image Version
    az sig image-version create -g $RESOURCE_GROUP --gallery-name MyGallery --gallery-image-definition centosimagedef --gallery-image-version 1.0.0 --managed-image {resource-id of your generalized image} --target-regions $REGION=1=standard_lrs --target-region-encryption MyDiskEncryptionSet --no-wait

#### Azure CycleCloud
Provision a CycleCloud from Azure Marketplace. Pick the latest version.
You can create a VM without a public IP so that this VM is not exposed to public internet.

Follow the steps below to provision a CycleCloud VM

https://learn.microsoft.com/en-us/azure/cyclecloud/qs-install-marketplace?view=cyclecloud-8

#### Azure Bastion
Azure Bastion is a fully managed service that lets you connect to  virtual machines using your browser and the Azure portal, or via the native SSH or RDP client already installed on your local computer. It needs a dedicated subnet and should be named as AzureBastionSubnet

    az network vnet subnet create -g $RESOURCE_GROUP --vnet-name MyVnet -n AzureBastionSubnet

Follow the steps mentioned in this document
    
https://learn.microsoft.com/en-us/azure/bastion/tutorial-create-host-portal





