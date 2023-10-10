The following document is onboard EDA workloads in your Azure subscription

##### Prerequisites

Have a new subscription with Owner/Contributor privileges



##### Getting Started
Head over to Azure Portal https://portal.azure.com
Click on Azure Cloud Shell Icon

#### Create a new resource group
az group create -l $REGION -n $RESOURCE_GROUP

#Create a new VNet to deploy compute/storage etc.
az network vnet create -l $REGION -g $RESOURCE_GROUP -n $VNET_NAME --address-prefix 10.0.0.0/16 --subnet-name MySubnet --subnet-prefixes 10.0.0.0/24

#Base Image
Create a new virtual machine from Azure Marketplace (CentOS, Ubuntu etc..) 
Install your custom tools/software. Follow the steps mentioned in this document to generalize your VM
https://learn.microsoft.com/en-us/azure/virtual-machines/generalize

#Create an Azure Key Vault and create your custom Keys as well.

#Create a new Storage account. 

Depending on your organization needs you can make the Azure KV and Storage Account private link enabled.
Follow these steps  if you need create a Storage Account with Customer Managed Keys.
https://learn.microsoft.com/en-us/azure/storage/common/customer-managed-keys-configure-new-account?tabs=azure-portal

Azure Netapp Files (ANF) is recommended to run HPC workloads. We need a dedicated subnet to provision ANF. 
Follow the below document to provision ANF. Make sure ANF is provisioned in the same the resource group, region and VNet.
https://learn.microsoft.com/en-us/azure/azure-netapp-files/azure-netapp-files-quickstart-set-up-account-create-volumes?tabs=azure-portal

#Create a Disk Encryption Set
az disk-encryption-set create --resource-group $RESOURCE_GROUP --name MyDiskEncryptionSet --key-url {Vault URI} --source-vault MyVault

#Create a Compute Image Gallery to store and version the generalized images
https://learn.microsoft.com/en-us/cli/azure/sig?view=azure-cli-latest#az-sig-create()

#Create Image Gallery
az sig create -g $RESOURCE_GROUP --gallery-name MyGallery -l $REGION

#Create Image Definition
az sig image-definition create -g $RESOURCE_GROUP --gallery-name MyGallery --gallery-image-definition centosimagedef --publisher KA --offer KA --sku KA --hyper-v-generation V2 --os-type linux --os-state Generalized

#Create Image Version
az sig image-version create --resource-group rg-intel-wus3 --gallery-name intelsig --gallery-image-definition centosimagedef \
--gallery-image-version 1.0.0 --managed-image {resource-id of your generalized image} 
--target-regions $REGION=1=standard_lrs --target-region-encryption intel-des1 --no-wait






