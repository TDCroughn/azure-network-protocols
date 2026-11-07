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

- Windows 11 (215H2)
- Ubuntu Server 24.04

<h2>High-Level Steps</h2>

## Part 1: Create the Virtual Machines

**1. Create a resource group**
1. Sign in at `https://portal.azure.com`.
2. In the search bar, type `Resource groups` and select it, then click **Create**.
3. Set **Subscription** to Subscription 1, **Resource group name** `RG-Network-Activities`, and **Region** to `(US) East US 2`.
4. Click **Review + create**, then **Create**.

*Expected result: the resource group appears in your Resource groups list.*

<p align="center">
<img src="Resource Group Created.png" width="80%" alt="Creating the resource group"/>
</p>

**2. Create the Windows 11 VM**
1. Search for `Virtual machines`, click **Create > Azure virtual machine**.
2. Basics tab: choose your resource group, name it `windows-vm`, set the region, and select the **Windows 11** image.
3. Choose a VM size (Standard_D2als_v6 - 2 vcpus, 4 GiB memory), set a username and password, and confirm the licensing box.
4. Networking tab: let Azure create a new **virtual network** and **subnet** (vnet-eastus2-1 (RG-Network-Activites), snet-eastus2-2 - 172.16.0.0/24).
5. Click **Review + create**, then **Create**.

*Expected result: deployment completes and `windows-vm` shows as Running.*

**3. Create the Ubuntu VM**
1. Click **Create > Azure virtual machine** again.
2. Basics tab: same resource group and region, name it `linux-vm`, select **Ubuntu Server**, and set **Authentication type** to **Password** with username `labuser`.
3. Networking tab: select the **same virtual network and subnet** that the Windows VM created.
4. Click **Review + create**, then **Create**.

*Expected result: both VMs are in the same virtual network. Confirm by opening each VM's **Networking** page, or open the virtual network's **Connected devices** list.*

<p align="center">
<img src="VMs in the Same Subnet.png" width="80%" alt="Both VMs in the same virtual network"/>
</p>

## Part 2: Observe ICMP Traffic

**4. Find the Ubuntu VM's private IP**
1. In the Azure portal, open `linux-vm`.
2. On the **Overview** page, locate the **Private IP address**: `[IP]`.

<p align="center">
<img src="Linux Private IP Address.png" width="80%" alt="Private IP address of linux-vm"/>
</p>

**5. Connect to the Windows VM with Remote Desktop**
1. Open `windows-vm` in the portal and click **Connect > RDP**.
2. Then sign in with the username and password you created.

*Expected result: the Windows 11 desktop opens in the Remote Desktop window.*

**6. Install Wireshark**
1. Inside the VM, open a browser and go to `https://www.wireshark.org`, then download the Windows installer.
2. Run the installer, accept the defaults (including Npcap), and finish the setup.

*Expected result: Wireshark opens from the Start menu.*

**7. Capture ICMP traffic to the Ubuntu VM**
1. Open Wireshark and double-click the **[adapter name]** interface to start a capture.
2. In the display filter bar, type `icmp` and press **Enter**.
3. Open Command Prompt and run `ping [linux-vm private IP]`.

*Expected result: the command prompt shows four replies, and Wireshark lists matching **Echo (ping) request** and **Echo (ping) reply** pairs between `[windows-vm IP]` and `[linux-vm IP]`.*

<p align="center">
<img src="05-icmp-ping-linux-vm.png" width="80%" alt="Wireshark showing ICMP echo requests and replies to the Ubuntu VM"/>
</p>

**8. Ping a public website**
1. In the command prompt, run `ping www.google.com`.

*Expected result: [what you saw, e.g. replies from Google's IP, and the matching ICMP entries in Wireshark].*

<p align="center">
<img src="06-icmp-ping-google.png" width="80%" alt="Wireshark showing ICMP traffic to a public website"/>
</p>

**What I observed:** [1-2 sentences, e.g. each ping produces one request and one reply, and the
reply source is the Ubuntu VM's private IP, which confirms both VMs can reach each other inside the virtual network.]
<br />
