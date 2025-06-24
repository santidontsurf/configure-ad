<p align="center">
<img src="https://i.imgur.com/pU5A58S.png" alt="Microsoft Active Directory Logo"/>
</p>

<h1>On-premises Active Directory Deployed in the Cloud (Azure)</h1>
This tutorial outlines the implementation of on-premises Active Directory within Azure Virtual Machines.<br />


<h2>Environments and Technologies Used</h2>

- Microsoft Azure (Virtual Machines/Compute)
- Windows App (Apple App Store)
- Active Directory Domain Services
- PowerShell

<h2>Operating Systems Used </h2>

- Windows Server 2022
- Windows 11 Pro (24H2)

<h2>High-Level Deployment and Configuration Steps</h2>

- Step 1: Preparing Active Directory Infrastructure in Azure
- Step 2: Deploying Active Directory
- Step 3: Creating users with Powershell
- Step 4: Managing Group Policy and Accounts

<h2>Overview</h2>
<p>For this tutorial, we will create a resource group in which we will put two virtual machines in the same virtual network. One VM will be our Active Directory domain controller running Windows Server and the other VM our client machine running Windows 11 Pro. We will join our Client-1 computer into the domain, then it will be aware of all the accounts in the domain and we'll be able to use one of those accounts to log in as Client-1 and simulate an environment like one you'd find working for an organization that utilized Active Directory.
In this case
</p>
<br/>
<p>In order for our Client-1 to join the domain, we need to have it use the DC-1 as it's domain server instead of the default VNET DNS server that it uses in Azure. To do so, we'll set Client-1's IP address and interface card to be the same as the IP address of DC-1. The reason we do this is because the deault VNET DNS server won't be able to direct us to the DC-1 domain when we ask it to, so we need to direct Client-1 in the right direction. </p>
<p align="center">
  <img src="https://i.imgur.com/d22FHIm.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
</p>
<br />
<h2>Preparing Active Directory Infrastructure in Azure</h2>
<p>First, we will go into Microsot Azure and create a resource group called "Active-Directory-Lab". Then create a virtual network called "Active-Directory-VNet" making sure that it's under the Active-Directory-Lab resource group and in the same region.With our resource group and virtual network deployed, we can now move on to create our two virtual machines.</p>
<br />
<p>We'll first create our DC-1 virtual machine and call it "dc-1", making sure it's under the Active-Directory-Lab resource group and under the same region. For it's image choose "Windows Server 2022" and that you create a username and password you can easily remember. Make sure to click both licensing boxes at the bottom. Hit next, and under the Networking tab make sure that we pick the Active-Directory-VNet, then finish creating the virtual machine, once it's created it should look like this:</p>
<p align="center"><img alt="Screenshot 2025-06-24 at 11 30 05 AM" src="https://github.com/user-attachments/assets/2c8d6241-d114-4920-b034-8ebf8e8fb90f" height="80%" width="80%"/>
</p>
<br />
<p>For the client virtual machine we will go through the same process by choosing the same resource group and virtual network. Make sure the VM is called "client-1" and that the image you choose is "Windows 11 Pro" and create a username and password you can remember. Once it's created it should look like this:</p>
<p align="center"><img alt="Screenshot 2025-06-24 at 11 31 34 AM" src="https://github.com/user-attachments/assets/51cb947a-9f6f-458f-b1e8-e71ad2da2974" height="80%" width="80%"/>
</p>
<br />
<p>After the virtual machiens are created, we are going to set DC-1's NIC private IP address to be static. The reason we do is becuse we don't want DC-1's address to change while it's being used as a DNS server for client-1. To do this, go to dc-1 and on the left, under "Networking", click "Network settings". There, click on the green interface icon:
</p>
<p align="center"><img alt="Screenshot 2025-06-24 at 11 42 33 AM" src="https://github.com/user-attachments/assets/7119b4c0-3131-4ab4-9312-18661d7c8afc" height="80%" width="80%"/>
</p>
<br />
<p>This opens the IP configurations where we can click on "ipconfig1" at the bottom, which opens a new window on the right of the page where we can see "Private IP address settings" and change Allocation from "Dyanmic" to "Static."</p>
<p align="center">
  <img alt="Screenshot 2025-06-24 at 11 45 27 AM" src="https://github.com/user-attachments/assets/f89958fb-1658-48db-a623-fec0faf17e9e" height="80%" width="80%" />
</p>
<br />
<p>Next, we're going to log in to dc-1 and disable firewall configurations. In a real world scenario this is not recommended, but for the sake of this tutorial we need to do this. </p>













