---
draft: true
date: 2025-02-10
categories:
  - Azure
tags:
  - azure front door  
---
# Building Windows VM Templates for Proxmox Using Packer
Even before the original buyout by Broadcom, VMware’s place in the hearts and minds of many virtualisation engineers had been waning. For some that later turned into panic, with alternatives being considered. One that has been mentioned a lot is Proxmox, with Google’s search results implying a large increase in interest in the last 12 months.

![Image](../media/proxmox.png)

<!-- more -->
In a large virtualisation estate there’s more to things than just spinning up a few virtual machines. One of the tasks that arises is the need to create templates of virtual machines. Typically these are regularly updated with security updates and have some basic configuration. The main tool for creating these images is Packer.

## Introducing Packer
Packer is a tool from Hashicorp that allows the creation of virtual machine images for a wide range of target platforms. The configuration of the virtual machine is defined in code. It supports on-prem virtualisation platforms such as VMware, Hyper-V and Proxmox as well as cloud providers such as AWS and Azure.

## Hardware Support Issues
Proxmox, like VMware, presents virtualised hardware to the operating system. And just like VMware, it can present a “safe” option that has broad driver support. For network cards, this is the Intel E1000. These safe options aren’t the high performance ones, so to get the maximum performance. The problem that arises is Windows doesn’t have out of the box support for these high performance options. This results in the installation not being able to see storage, causing the installation to halt early on.

In the case of Proxmix, these high performance (so-called “paravirtualised”) devices require VirtIO drivers. These can be provided to the Windows installation mounting the install ISO, and specifying the path to the drivers. When done manually, it appears that only the storage driver is install, leaving other devices non-functional. The key one of those is the network interface.

The approach taken by some examples I had seen was to specify the path for the storage driver and then later, run a script that would install the remaining drivers and guest tools.

## Packer File
In following the other approachs I found online, I was able to create a packer file that did manage to build the image through to completion.

## Other Issues
I ran into a number of other issues in the process of getting the Packer file to work. Below are some highlights:
