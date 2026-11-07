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
1. Open Wireshark and double-click the **ethernet** interface to start a capture.
2. In the display filter bar, type `icmp` and press **Enter**.
3. Open Command Prompt and run `ping 172.16.0.5`.

*Expected result: the command prompt shows four replies, and Wireshark lists matching **Echo (ping) request** and **Echo (ping) reply** pairs between `172.16.0.4` and `172.16.0.5`.*

<p align="center">
<img src="ICMP Ping From Windows to Linux.png" width="80%" alt="Wireshark showing ICMP echo requests and replies to the Ubuntu VM"/>
</p>

**8. Ping a public website**
1. In the command prompt, run `ping www.google.com`.

*Expected result: [what you saw, e.g. replies from Google's IP, and the matching ICMP entries in Wireshark].*

<p align="center">
<img src="ICMP Ping from Windows to Google.com.png" width="80%" alt="Wireshark showing ICMP traffic to a public website"/>
</p>

**What I observed:** Each ping produced one ICMP echo request from the Windows VM
(172.16.0.4) and one echo reply from the Linux VM (172.16.0.5), for a total of
4 requests and 4 replies. The two VMs can communicate because they share a virtual
network and subnet, and Azure's default NSG rules allow traffic between resources in
the same virtual network. I also compared the MAC addresses in the Ethernet II header
with `ipconfig /all` on Windows and `ip link` on Linux to verify that they showed up in Wireshark.  

## Part 3: Block and Restore ICMP with a Network Security Group

**9. Start a continuous ping**
1. In the Windows VM, open Command Prompt and run `ping 172.16.0.5 -t`.
2. Keep Wireshark running with the `icmp` filter.

*Expected result: the command prompt shows steady replies and Wireshark shows request/reply pairs.*

<p align="center">
<img src="Non-Stop Ping From Windows to Linux.png" width="80%" alt="Continuous ping with replies from linux-vm"/>
</p>

**10. Block inbound ICMP on the Ubuntu VM's NSG**
1. In the Azure portal, open `linux-vm` and click **[Networking / Network settings]**.
2. Click **Add inbound port rule** and enter: Source `Any`, Source port ranges `*`, Destination `Any`, Service `Custom`, Destination port ranges `*`, Protocol `ICMP`, Action `Deny`, Priority `290`, Name `DenyICMP`.
3. Click **Add**.

*Expected result: the new deny rule appears in the inbound rules list above the default rules.*

<p align="center">
<img src="Deny ICMP Rule.png" width="80%" alt="Creating the inbound deny rule for ICMP"/>
</p>

**11. Observe the blocked traffic**
1. Return to the Windows VM and watch the ping window and Wireshark.

*Expected result: [what you saw, e.g. after about a minute the ping shows "Request timed out" and Wireshark shows echo requests with no replies].*

<p align="center">
<img src="Ping Stopped by IMCP Deny Rule.png" width="80%" alt="Ping timing out and Wireshark showing requests without replies"/>
</p>

**12. Restore ICMP and stop the ping**
1. Back in the NSG, [delete the deny rule / change its action to Allow].
2. Return to the Windows VM and watch the ping window and Wireshark.

*Expected result: [what you saw, e.g. replies resume in the command prompt and Wireshark].*

3. Press **Ctrl+C** in the command prompt to stop the ping.

<p align="center">
<img src="Ping Restarts After Deny Rule is Deleted.png" width="80%" alt="Ping replies resuming after the NSG rule was removed"/>
</p>

**What I observed:** [your own explanation, see below]

## Part 4: Observe Network Protocol Traffic

**13. Observe SSH traffic**
1. In the Windows VM, open Wireshark and start a capture, then set the display filter to `ssh`.
2. Open PowerShell and run `ssh labuser@[linux-vm private IP]`, accept the host fingerprint prompt, and enter the password.
3. Run `[commands you typed]`, then type `exit` and press **Enter**.

*Expected result: SSH packets appear in Wireshark as soon as the connection starts and with every command typed. [What you saw, e.g. the payload is encrypted, so the commands themselves aren't readable.]*

<p align="center">
<img src="Traffic from SSH.png" width="80%" alt="Wireshark showing SSH traffic between the VMs"/>
</p>

**14. Observe DHCP traffic**
1. In Wireshark, change the display filter to `dhcp`.
2. Open PowerShell as administrator and run `ipconfig /renew`.

*Expected result: [what you saw, e.g. DHCP Request and ACK packets between the VM and the DHCP server at [IP]].*

<p align="center">
<img src="12-dhcp-traffic.png" width="80%" alt="Wireshark showing DHCP traffic after ipconfig /renew"/>
</p>

**15. Observe DNS traffic**
1. In Wireshark, change the display filter to `dns`.
2. In a command prompt, run `nslookup google.com` and `nslookup disney.com`.

*Expected result: [what you saw, e.g. a DNS query and response for each name, with the resolved IP addresses and the DNS server used].*

<p align="center">
<img src="13-dns-traffic.png" width="80%" alt="Wireshark showing DNS queries and responses for google.com and disney.com"/>
</p>

**16. Observe RDP traffic**
1. In Wireshark, change the display filter to `tcp.port == 3389`.

*Expected result: [what you saw, e.g. a continuous stream of packets even with no activity].*

<p align="center">
<img src="14-rdp-traffic.png" width="80%" alt="Wireshark showing continuous RDP traffic on port 3389"/>
</p>

**What I observed:** [your own explanation, see below]
<br />
