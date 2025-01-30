---
draft: false
date: 2015-10-13
categories:
  - VMware
tags:
  - vmware
---
# vCloud Air Test Experience
vCloud Air is VMware’s public cloud offering, similar to Amazon’s AWS or Microsoft’s Azure. The key distincion between vCloud Air and these other offerings is that vCloud Air uses VMware’s products such as vSphere.

The VMWare User Group (VMUG) recently added free credits on vCloud Air OnDemand as part of their EVALExperience program. As the name suggests, vCloud Air OnDemand is a pay-as-you-go service. I looked at this service offering as a server engineer with a reasonable background in VMware, considering aspects such as the ease of basic tasks, general administration, technical considerations for the business (good and bad) and how it compares to other offerings.
<!-- more -->

## Signup Process
The signup process was very easy and allows you to attach the service to an existing VMware account. Once provisioned, it will appear under your MyVMware Subscription Services. This area allows you to view current or past usage, as well as details relating to the service such as payment options.

## Region Selection
Like AWS and Azure, the OnDemand service has regions so you’re prompted to select a geographical region to store your Virtual Machines when you initiate it. At the time of writing, the options are:

* Australia
* Germany
* Japan
* UK
* 3 US locations – California, Texas and Virginia

This is a reasonably good geographical dispersion for the first iteration of this product. The Australia region is located in Sydney so latency should be tolerable for even West Coast administrators.

After you select your location, your service will be provisioned, taking a couple of minutes. Once it’s ready, it will present an option to create your first virtual machine.

## Creating Virtual Machines
The New Virtual Machine wizard presents VMware’s catalog of templates, as well as allowing you to utilise your own or create a Virtual Machine from scratch. It’s worth noting that the Windows-based templates attract a licensing fee in addition to their general running costs (at the time of writing, this is costed at 9.6 cents per vCPU per hour).

![Image](../media/2015-10-13-001.jpg)

For my first Virtual Machine, I used the Windows 2012 R2 template. After selecting the template, settings such as RAM, CPU, storage and the name can be tweaked. A cost per hour is displayed, giving an idea of the running cost (this can be changed to a per month display as well). An interesting setting is by default, the CPU and RAM amounts are linked, creating a recommended range of RAM depending on the vCPUs. A 1 vCPU setting will have a 2-16GB of RAM range while for 2 vCPUs the range is 4 to 32GB of RAM.

A random password is set when the Virtual Machine is provisioned, which is hidden in the Virtual Machine’s details under the Guest OS heading. The Virtual Machine’s console which doesn’t appear to require any plugins, which is a plus.

## Managing Virtual Machines
The Actions menu for Virtual Machines contains the usual options (Power On/Off, Reset) as well as Change Owner, Edit Resources and Connect to Internet.

The Edit Resources is an interesting option because it allows an administrator to adjust a running virtual machine’s resources upwards on the fly. As shown below, I changed the memory from 4GB to 6GB without needing to restart the virtual machine. When I dug into the settings for my Virtual Machine in the vCloud Director admin interface, I could see that the Virtual Machine was configured to allow vCPU and RAM hot add.

![Image](../media/2015-10-13-002.jpg)

By default, Virtual Machines in vCloud Air are on a private IP range and aren’t directly connected to the Internet. This is a distinct difference from AWS or Azure. The Connect to Internet action allows you to expose your virtual machine to the internet, but requires a public IP be assigned to your gateway, which you are charged for. This default setup and approach to getting a virtual machine on the internet may seem counter-intuitive to those more familiar with AWS or Azure but I think it fits in line with the customers VMware is targeting (which I’ll go into more detail later).

## Digging Deeper Into Administration
vCloud Air has what could be called two administrative interfaces – there is one for basic admin tasks, but from time to time you will see a “Manage in vCloud Director” link when looking at an object. Clicking on this will launch a vCloud Director instance for managing your settings. In this interface you can get into really advanced options for managing Virtual Machines, Networks and so on. For anyone who’s used vSphere 5.5 or 6.0’s web interface, this will probably be a lot more familiar than the default administrative UI.

## vCloud Connector
Earlier I mentioned what I think VMware’s target audience is for this product. Taking a page out of Microsoft’s book in relation to tight administrative integration, VMware have produced vCloud Connector, which allows you to connect your private clouds and public clouds to allow central administration.

![Image](../media/2015-10-13-003.jpg)

This means in effect your organisation uses the same tool set for managing resources in private and public clouds, use a common set of templates across these clouds and remove the need for specialist knowledge in your IT staff.
To get the vCloud Connector system working, you need to setup two Virtual Appliances, the vCloud Connector Server (vCCS) and the vCloud Connector Node (vCCN).

Once these are setup and configured to link to your clouds and your vCenter instance, you can manage private and public clouds from a single unified console.

![Image](../media/2015-10-13-004.jpg)

This setup also allows two new features – Stretched Deploy and Offline Data Transfer. A Stretched Deploy is essentially moving a Virtual Machine from your in-house cloud to your public cloud. The process creates a network tunnel between the clouds so the Virtual Machine can still communicate with your in-house environment. Offline Data Transfer allows you to transfer groups of powered-off Virtual Machines or Templates to vCloud Air.

## API Support
Another point of interest for me was vCloud Air’s API support, specifically Powershell. The API support from a programming point of view seems reasonably developed, with support for manipulating users, creating virtual machines and so on. At the time of writing, the latest update I could find in relation to Powershell support was that the vCloud Air PowerCLI module was still in evaluation and doesn’t appear to have support for vCloud Air OnDemand yet.

## Access Management
At the time of writing, vCloud Air by itself doesn’t support integration with Active Directory which means adding access on the web interface is a manual process. I have seen some information that suggests if you use vRealize Automation (vRA), you can use vRA’s Active Directory integration, as well as future plans to include Active Directory integration with vCloud Air.

## Final Thoughts
From what I’ve seen so far, it seems clear that VMware is going for a particular vision and selling point with their vCloud Air product. By using the same virtualisation technology that most organisations already use in-house, they have removed a major roadblock for customers to move into a cloud environment. No time consuming messy conversion process required, just transfer over. By positioning the vCloud Connector and vRA as the fulcrum between in-house clouds and vCloud Air, have reduced the requirement for much, if any, staff training. The Powershell support needs another iteration or two to get to a decent mature stage, but it’s an interesting product for a first release.
