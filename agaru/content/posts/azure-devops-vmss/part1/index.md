+++
title = "Azure Devops & VMSS - part1"
date = "2021-03-05"
description = "An introduction"
+++

# Azure DevOps & VMSS agents
![farnsworth.png](farnsworth.png)
Good news everyone! Azure DevOps now supports VMSS as agent.

In this series of posts, we'll cover:
* Why use a VMSS with Azure DevOps?
* How to create and configure a simple VMSS.
* Customizing the VMSS.
* Use Microsoft's script to create an image for our VMSS.
* Getting freaky with those images.
* Security aspects.

## Why use VMSS with Azure DevOps?
### Private resources
Unless you're a startup or have committed to the cloud 200%, you're likely to have private resources like IIS servers in an internal zone, an instance of SonarQube CE that you do not want to expose on the internet ever (why would you even do that?), some ESB/EAI you'd like to use during the build to run integration tests instead of mocking your services,...

On the plus side, Azure has started to offer [private endpoints](https://docs.microsoft.com/en-us/azure/private-link/private-endpoint-overview) for a lot of resources. This is another topic, but the big shift is that you can now put most of your PaaS in a permitter (aka VNet) instead of going full public.

For all these reasons, DevOps' hosted agents won't do. They run *somewhere* and you don't want to expose all yours precious private artefacts.

Since the pipelines needs a runtime, your only choice was to use self hosted agent and you might not find this approach good enough : maintenance, scalability and price aren't that great.

### VMSS vs self hosted agents
As you might or not know, Azure DevOps allows you to create pipelines to build, test, transform, deploy,... And those pipelines were either running in the cloud with hosted agents, managed & hosted by Microsoft, or on private VMs known as self hosted agents.
For those, you had to 
* install the VM
* install the tools needed by the pipeline (such as Docker for example)
* install the agent
* maintain the VM (OS & tool upgrade)
* install a second VM

With the VMSS, it's gonna makes things a little bit easier. For simple use case, a naked Ubuntu will do, and this will makes maintenance a breeze. Give the VMSS to DevOps a let it do its magic.
The agent will automatically be installed and configured, the number of instances will increase/decrease based on 3 simple rules : max number of instances, min number of instance(s), idle time before killing an instance.

For more advanced user, we'll later see how to use the packer template files of Microsoft to create image matching the offer of the managed host in your own private VMSS. Can you imagine: cheaper, easy to maintain and private managed instance inside your VNet? 🤩

### Cost
For simplification, I will round up the prices.
Microsoft Hosted agents are 33,7€ a month, first one is free for 1800minutes. Let's say you have quite a lot of pipelines to run and you want 10 of them; that will be 337€ a month, wether you use them or not. You pay for a ticket, it's up to you to show up.

Now, we'll add 4 self hosted agents for release purposes on your private resources. You'll have to pay 38€ in DevOps + the cost of 4 VMS that will be running 24/24: 295,5€ for 4 Ubuntu D2 v3 (2vCPU & 8 GB RAM).

That's a grand total of: **337 + 295,5 + 38 : 670,5€**. 

Note that you pay the self hosted price of 12,65€ for each parallel job you are running and not for each agent. Plus, the first one is free and for each active Visual Studio Enterprise subscriber who is a member of your organisation, you get one additional self-hosted parallel job.

Of course, you can save costs by shutting down the VMs during the night and weekends, by challenging the devs to reduce the pool size,... but we all know those talks generates more talks and end up costing more money in meeting.

Now, let's see for a VMSS, is does not look like you have to pay anything for the // jobs but let's assume the price will be the same as the self hosted.

We'll make a pool of 0 to 10 VMs. Idle time has no meaning here, each situation will be different. The VM will be D2 v3 with an Ubuntu OS.

If the build are running 24/24 7/7, you'll have to pay 113.85€ in DevOps for the // self hosted builds and 738.78€ of VM in the sale set: 852,63€. If no jobs are running, you'll only have to pay the 113,85€.

In a more realistic scenario, let's assume people are making build and release 10h a day during the weekdays only. We'll have 4 days of heavy build / test / release (end of the sprint) each month. 5 VMs will be on during regular days and 8 during heavy load: **113,85 + 303,59 (regular use) + 12,14 (peak use): 419€**.

Note that those are only estimates. You'll have to take a lot into account to have a precise comparaison.

### Custo!
Self hosted agents aren't customisable. You can't change the default java version for all pipelines, you can't add tools or preinstall dependencies.
Of course, you can always add tasks in your pipelines to do that but that will makes your yaml crowded and complex (you're using yaml for you pipelines, right?) and the task needs to run each time you make a build.

Self hosted can be tailored to your need, after all, it's your little VM. But who has time for maintenance. Plus, you know too well you'll have to upgrade a framework at the worst possible time (Friday afternoon, the release must happen but the agent has to be patched first).

## Next!
Come back and see how to create the VMSS and link it to your Azure DevOps!