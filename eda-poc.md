The following document is onboard EDA workloads in your Azure subscription

#Prerequisites
1) Have a new subscription with Owner/Contributor privileges



Head over to Azure Portal https://portal.azure.com
Click on Azure Cloud Shell Icon

Create a new resource group
az group create -l $REGION -n $RESOURCE_GROUP

 Create a new VNet to deploy compute/storage etc.
az network vnet create -l $REGION -g $RESOURCE_GROUP -n $VNET_NAME --address-prefix 10.0.0.0/16 --subnet-name MySubnet --subnet-prefixes 10.0.0.0/24

Create a new virtual machine from Azure Marketplace (CentOS, Ubuntu etc..) 
Install your custom tools/software. Follow the steps mentioned in this document to generalize your VM
https://learn.microsoft.com/en-us/azure/virtual-machines/generalize

Create an Azure Key Vault and create your custom Keys as well.

Create a new Storage account. 

Depending on your organization needs you can make the Azure KV and Storage Account private link enabled.
Follow these steps  if you need create a Storage Account with Customer Managed Keys.
https://learn.microsoft.com/en-us/azure/storage/common/customer-managed-keys-configure-new-account?tabs=azure-portal

Azure Netapp Files (ANF) is recommended to run HPC workloads. We need a dedicated subnet to provision ANF. 
Follow the below document to provision ANF. Make sure ANF is provisioned in the same the resource group, region and VNet.
https://learn.microsoft.com/en-us/azure/azure-netapp-files/azure-netapp-files-quickstart-set-up-account-create-volumes?tabs=azure-portal

Create a Compute Image Gallery to store and version the generalized images
https://learn.microsoft.com/en-us/cli/azure/sig?view=azure-cli-latest#az-sig-create()

az sig create -g $RESOURCE_GROUP --gallery-name MyGallery -l $REGION

az sig image-definition create -g $RESOURCE_GROUP --gallery-name MyGallery --gallery-image-definition centosimagedef --publisher KA --offer KA --sku KA --hyper-v-generation V2 --os-type linux --os-state Specialized




