+++
title = "Azure Devops & VMSS - part2"
date = "2021-03-11"
description = "Creating the VMSS"
+++

# Azure DevOps & VMSS agents
Good news everyone! Azure DevOps now supports VMSS as agent.

In this series of posts, we'll cover:
* ~~Why use a VMSS with Azure DevOps?~~
* How to create and configure a simple VMSS.
* Customising the VMSS.
* Use Microsoft's script to create an image for our VMSS.
* Getting freaky with those images.
* Some security aspects.

## Azure CLI
First, let's [install Azure CLI](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli). 

## Create the VMSS
We can now create the VMSS. This is super simple!

For a VMSS, you'll need :
* An Azure Subscription
* A VNet
* A dedicated subnet (optional but recommended)

Get your subnet ID by adding /subnets/nameofyoursubnet after the ID of the VNet of by using Azure CLI:
```
vnetRgName=NAME_OF_THE_RG_VNET
vnetName=NAME_OF_THE_VNET
subnetName=NAME_OF_THE_SUBNET
subnetId=$(az network vnet subnet list -g $vnetRgName --vnet-name $vnetName --query "[?contains(name, '$subnetName')].id" -o tsv)
```

Now, set a default username, a name for your VMSS, a resource group (it needs to exist before this command) and the subnet ID you got just before.
```
user=YOUR_USERNAME
vmssName=A_NAME_FOR_VMSS
rgName=NAME_OF_THE_RG
subnetId=ID_OF_YOUR_SUBNET
az vmss create \
--admin-username $user \
--authentication-type SSH \
--disable-overprovision \
--image UbuntuLTS \
--instance-count 2 \
--load-balancer "" \
--name $vmssName \
--nsg "" \
--platform-fault-domain-count 1 \
--public-ip-address "" \
--resource-group $rgName \
--single-placement-group false \
--storage-sku Standard_LRS \
--subnet $subnetId \
--upgrade-policy-mode manual \
--vm-sku "Standard_D2_v3"
```

And voilà, you have your VMSS. You need to make sure this VMSS can access internet on 80/443 TCP and you're good to go. We'll talk security later.

## Link it to Azure DevOps
Go to you Azure DevOps portal, at the root (not in a project) and go to the organization settings.

Now, in the left menu, select Agent pools under Pipelines and click on *Add pool*.

Select *Azure virtual machine scale set* as type, select a projet (this is needed to store the SPN, see this projet like to mother projet of the scale set. You can the project were you're gonna store the code of this lab), the Azure subscription where the scale set has been created and finally, the scale set.

Now, you can give it a name, put the min/max number of instances and the idle time. The 3 values can be changed later on.

## All done
Azure DevOps is now the owner of the scale set. An extension will be added to install and configure the agent every time an instance is fired up. If you go on your VMSS, you'll the first 2 instances getting deleted and 3 new ones being provisioned. They be killed after the idle time so, do not worry too much about the cost here.