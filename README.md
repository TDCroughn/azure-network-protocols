# azure-network-protocols

<p align="center">
<img src="https://i.imgur.com/Ua7udoS.png" alt="Traffic Examination"/>
</p>

<h1>Network Security Groups (NSGs) and Inspecting Traffic Between Azure Virtual Machines</h1>
In this tutorial, we observe various network traffic to and from Azure Virtual Machines with Wireshark as well as experiment with Network Security Groups. <br />


<h2>Environments and Technologies Used</h2>

- Microsoft Azure (Virtual Machines/Compute)
- Remote Desktop
- Various Command-Line Tools
- Various Network Protocols (SSH, RDH, DNS, HTTP/S, ICMP)
- Wireshark (Protocol Analyzer)

<h2>Operating Systems Used </h2>

- Windows 10 (21H2)
- Ubuntu Server 20.04

<h2>High-Level Steps</h2>

## Part 1: Create the Virtual Machines

**1. Create a resource group**
1. Sign in at `https://portal.azure.com`.
2. In the search bar, type `Resource groups` and select it, then click **Create**.
3. Set **Subscription** to Subscription 1, **Resource group name** `RG-Network-Activities`, and **Region** to `(US) East US 2`.
4. Click **Review + create**, then **Create**.

*Expected result: the resource group appears in your Resource groups list.*

<p align="center">
<img src="01-resource-group.png" width="80%" alt="Creating the resource group"/>
</p>

**2. Create the Windows 10 VM**
1. Search for `Virtual machines`, click **Create > Azure virtual machine**.
2. Basics tab: choose your resource group, name it `windows-vm`, set the region, and select the **Windows 10** image.
3. Choose a VM size ([size]), set a username and password, and (if offered) confirm the licensing box.
4. Networking tab: let Azure create a new **virtual network** and **subnet** ([name], [address range]).
5. Click **Review + create**, then **Create**.

*Expected result: deployment completes and `windows-vm` shows as Running.*

**3. Create the Ubuntu VM**
1. Click **Create > Azure virtual machine** again.
2. Basics tab: same resource group and region, name it `linux-vm`, select **Ubuntu Server**, and set **Authentication type** to **Password** with username `labuser`.
3. Networking tab: select the **same virtual network and subnet** that the Windows VM created.
4. Click **Review + create**, then **Create**.

*Expected result: both VMs are in the same virtual network. Confirm by opening each VM's **Networking** page, or open the virtual network's **Connected devices** list.*

<p align="center">
<img src="02-vms-same-vnet.png" width="80%" alt="Both VMs in the same virtual network"/>
</p>
<br />
