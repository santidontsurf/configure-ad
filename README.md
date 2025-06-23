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

<h2>Preparing Active Directory Infrastructure in Azure</h2>
<p>In this lab we will create two VMs in the same VNET. One will be a Domain Controller, the other will be a Client machine. We will change the DC to a static IP because its offering Active Directory services to the client machine. Client machine will be joined to the domain. We will control the DNS settings on the client machine, the client machine will use the DC as its DNS server.</p>
<p align="center">
  <img src="https://i.imgur.com/d22FHIm.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
</p>
<br />
<p>First, we will </p>
