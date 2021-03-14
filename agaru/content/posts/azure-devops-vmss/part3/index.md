+++
title = "Azure Devops & VMSS - part3"
date = "2021-03-14"
description = "Customising the VMSS"
+++

# Azure DevOps & VMSS agents
Good news everyone! Azure DevOps now supports VMSS as agent.

In this series of posts, we'll cover:
* ~~Why use a VMSS with Azure DevOps?~~
* ~~How to create and configure a simple VMSS.~~
* Customizing the VMSS.
* Use Microsoft's script to create an image for our VMSS.
* Getting freaky with those images.
* Security aspects.

## Custo!

So far, we've taken the Ubuntu LTS image and fed it to a VMSS. By linking the scale set with Az DevOps in the previous post, we gave DevOps the ability to drive the scale set. It now manages the instances based on it's need and the constraints you've (min/max number of instance, idle time,... ) and it injects and configure the agent.

You basically have a naked Ubuntu image so, what if you want to build a docker container from a dockerfile? Sure, you have an [Install Docker CLI](https://docs.microsoft.com/en-us/azure/devops/pipelines/tasks/tool/docker-installer?view=azure-devops) pipeline task available but it'll only install the cli and not the engine.

You could also make a step in your pipeline to install the engine when the job is running but this will consume time and resources each time you run it.

You could also use a [script that'll run each time an instance is created](https://docs.microsoft.com/en-us/azure/devops/pipelines/agents/scale-set-agents?view=azure-devops#customizing-virtual-machine-startup-via-the-custom-script-extension) or [use cloud-init](https://docs.microsoft.com/en-us/azure/virtual-machines/linux/using-cloud-init) to attain the same job (tough you'll have to [delay walinuxagent start](https://github.com/Azure/WALinuxAgent/issues/1938#issuecomment-657293920)), but you'll pay the execution every time you instantiate an agent. 

On the plus side, your image is not immtuable, you lose control of your pipeline runtime and things might start to break without you being able to rollback anything.

## Custom image

### Create the VM

Here, we'll create a VM, install what we need and generate a managed image that we'll be able to use in our scale set.

First, let's create a VM. We'll reuse the rg and subnet created in the [previous post]({{< ref "/content/posts/azure-devops-vmss/part2/index.md" >}}):

```bash
user=YOUR_USERNAME
rgName=NAME_OF_THE_RG
vmName=vmssTemplate
az group create --name $rgName --location westeurope
az vm create \
    --authentication-type SSH \
    --resource-group $rgName \
    --name $vmName \
    --image UbuntuLTS \
    --subnet $subnetId \
    --admin-username $user 
```

Ii you're unsure how to get the subnet id, read the previous post.

Once your VM is up and running, ssh to it and start doing what you need. For the example, here's how to install the docker engine:

```bash
# update & upgrade
sudo apt-get update 
sudo apt-get upgrade
# add needed packages
sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg \
    lsb-release

# get the repo key
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

# add the repo
echo \
  "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# update & install!
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io
```

Now you're done installing your things. It's time to test the installation by running a hello world.

```bash
docker run hello-world
```

If things works out, you should see a *Hello from Docker!*. It is advised to test everything you've installed before going to the next step.

Remember, there's no need to install the agent, this will be done for you by Azure DevOps when linking the new scale set.

### Deprovision the VM

Let's remove the machine specific stuff and remove the last created user (the one you're using) by running `sudo waagent -deprovision+user`.

Wait the command to end and close your ssh session. Now, deallocate the VM and generalize it! 

```bash
az vm deallocate \
    --resource-group $rgName \
    --name $vmName
    
az vm generalize \
    --resource-group $rgName \
    --name $vmName
```

Your VM cannot be started anymore but we'll generate an image from it:

```bash
imageName=NAME_OF_THE_IMAGE
az image create \
    --resource-group $rgName \
    --name $imageName --source $vmName
```

Now you can follow the [previous part]({{< ref "/content/posts/azure-devops-vmss/part2/index.md" >}}) again, create a new scale set and link it to DevOps. Do not forget the change the **--image UbuntuLTS** by the id of your new image. You can list the images id with **az image list**. I don't want to scare you with the filtering... yet ;)

## Conclusion

Congratulation, you now have a custom agent scale set for your very own pipelines. But as a devops engineer, you probably have a much better use of your time than generating manual image to update the scale set. 

## Resources

[How to create a managed image of a virtual machine or VHD](https://docs.microsoft.com/en-us/azure/virtual-machines/linux/capture-image)

[Docker install on Ubuntu](https://docs.docker.com/engine/install/ubuntu/)