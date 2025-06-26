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
<p>Next, we're going to log in to dc-1 and disable firewall configurations. In a real world scenario this is not recommended, but for the sake of this tutorial we need to do this. Go into the dc-1 in Azure and get dc-1's public IP address and connect to it via either Remote Desktop (Windows) or Microsoft Remote Desktop (Mac).</p>
<br />
<p>Once in dc-1, if you configured dc-1 exactly as this tutorial recommended, your main page should've opened Server Manager like this:</p>
<p align="center"><img alt="Screenshot 2025-06-25 at 11 03 15 AM" src="https://github.com/user-attachments/assets/e576dfa4-1a9c-45a3-a49c-97171f57ea70" height="80%" width="80%"/>
</p>
<br />
<p>Now, right-click the start menu (Windows icon) in the bottom left and select "Run", a small window should open up. Type "wf.msc" to open Windows Defender Firewall with Advanced Security.</p>
<p align="center"><img alt="Screenshot 2025-06-25 at 11 06 16 AM" src="https://github.com/user-attachments/assets/86d8544f-07db-472d-8867-c22c9dbf9d6d" height="80%" width="80%" />
</p>
<br />
<p>Inside Windows Firewall, click "Windows Defender Firewall Properties", which opens a new page where you need to change the Firewall state from "on" to "off". Do the same for the Private profile and the Public profile as well, then click "OK".</p>
<p align="center"><img alt="Screenshot 2025-06-25 at 11 11 26 AM" src="https://github.com/user-attachments/assets/8944327c-2769-4851-8c66-a8a1d17462eb" height="80%" width="80%"/>
</p>
<br />
<p>Now that Firewall settings on dc-1 are disabled, we are going to change the DNS settings on client-1 to point to dc-1's private IP address. To do this, go to dc-1 on Azure and find it's private IP address under the Networking details, and copy that address.</p>
<p align="center"><img alt="Screenshot 2025-06-25 at 11 13 33 AM" src="https://github.com/user-attachments/assets/d644da37-e52e-40c4-a99e-315dcc202f0c" height="80%" width="80%"/>
</p>
<br />
<p>Then, travel to client-1 within Azure and into "Network settings" and click on the green interface icon, and right below "IP configurations on the left, click "DNS servers" and it should look like this:</p>
<p align="center"><img alt="Screenshot 2025-06-25 at 11 18 44 AM" src="https://github.com/user-attachments/assets/84882f3a-5186-416f-9bf5-382655588443" height="80%" width="80%"/>
</p>
<p>Right now, the DNS server preference is set to "Inheret from virtual network", but we are going to choose "custom" and paste dc-1's private IP address and click "Save" at the top. Now, client-1's DNS server points to dc-1's domain.</p>
<p align="center"><img alt="Screenshot 2025-06-25 at 11 23 47 AM" src="https://github.com/user-attachments/assets/4faaa8e3-8c7c-487b-acdc-275d4b940931" height="80%" width="80%" />
</p>
<br />
<p>In order for this change to be fully made, we need to restart the client-1 virtual machine before proceeding with the tutorial.</p>
<br />
<p>Next, we'll login to client-1 and open up Powershell. We want to ping dc-1's private IP address to see if our last configruation actually worked, so grab dc-1's private IP address from Azure and back in Powershell type "ping [dc-1 private IP address]" and hit enter. This should be what the result looks like:  </p>
<p align="center"><img alt="Screenshot 2025-06-25 at 11 36 04 AM" src="https://github.com/user-attachments/assets/55eeed1b-ac1d-4915-9655-c1a2068ae1da" height="80%" width="80%"/>
</p>
<br />
<p>If you got the same result, great! You've correctly configured everything so far. If you're ping command replies with anything else, like "Destination host unreachable" it means that dc-1 and client-1 are in different virtual networks or dc-1's Firewall is blocking ping.</p>
<br />
<p>The last thing we need to do on Powershell is check that the DNS server for client-1 reflects dc-1's private IP addreess. For this, type "ipconfig /all" then hit enter. Scroll down until you find "DNS server" and you should see dc-1's private IP address next to it.</p>
<p align="center"><img alt="Screenshot 2025-06-25 at 11 40 31 AM" src="https://github.com/user-attachments/assets/bbe59583-4b10-4fe4-a064-8b26bca14191" height="80%" width="80%"/>
</p>
<br />
<h2>Deploying Active Directory</h2>
<p>Now we'll download Active Directory Domain Services in the domain controller. Go back to the dc-1 virtual machine and open up Server Manager if it's not already open. Inside, click on "Add roles and features" and hit next on the new page until you reach "Server Roles". In here, make sure that you check "Active Directory Domain Services" at the top of the list and click "Add Feature" on the pop-up page. Continue until you reach "Confirmation".</p>
<p align="center"><img alt="Screenshot 2025-06-26 at 11 02 37 AM" src="https://github.com/user-attachments/assets/f4d34e87-148e-4c71-b422-696da9575289" height="80%" width="80%" />
</p>
<br />
<p>In the Confirmation page, make sure that "Restart the destination server automatically if required" is checked then click "Install" at the bottom. Once installed you can close the tab.</p>
<p align="center"><img alt="Screenshot 2025-06-26 at 11 04 36 AM" src="https://github.com/user-attachments/assets/f60818e1-c420-4b68-9d08-c316a698f992" height="80%" width="80%"/>
</p>
<br />
<p>So far, we've been configuring dc-1 as a domain controller without actually promoting it to a domain controller which is what we'll do next. In Server Manager at the top-right corner there's  a flag icon, click on it, then click "Promote this server to a domain controller".</p>
<p align="center"><img alt="Screenshot 2025-06-26 at 11 09 59 AM" src="https://github.com/user-attachments/assets/6b2dfc80-f84d-41b9-b3de-16c733b02c7c"  height="80%" width="80%"/>
</p>
<br />
<p>In Deployment Configuration, choose "Add a new forest" and type "mydomain.com" in the bar, then hit next.</p>
<p align="center"><img alt="Screenshot 2025-06-26 at 11 11 53 AM" src="https://github.com/user-attachments/assets/dab6856e-66f5-48e1-a966-efdc79ac2d50" height="80%" width="80%"/>
</p>
<br />
<p>In Domain Controller Options you'll be asked to set a password for your domain controller, make sure it's a password you can easily remember. Click "Next" and in DNS Options, make suer that "Create DNS delegations" is unchecked. Hit "Next" on the remaining tabs until you reach "Prerequisites Check" and if everything is configured correctly, hit "Install" at the bottom.</p>
<p align="center"><img alt="Screenshot 2025-06-26 at 11 19 22 AM" src="https://github.com/user-attachments/assets/360bd485-4246-402d-aa24-70b62cd67515" height="80%" width="80%"/>
</p>
<br />
<p>You'll be automatically logged out of the virtual machine after dc-1 is promoted to domain controller, and now that dc-1 no longer a regular virtual machine, to log back into it you need to specify if you're login in as a domain user or a local user. We want to login as a domain user now, so we need to add "mydomain.com/" to our username like this:</p>
<p align="center"><img alt="Screenshot 2025-06-26 at 11 31 19 AM" src="https://github.com/user-attachments/assets/10c52976-a7f3-4802-a329-b7e13e6724d7" height="80%" width="80%"/>
</p>
<br />
<p>Once we're in dc-1, we're going to create a domain admin user within our domain to be used for our administrative tasks. To do this, type "Active Directory Users and Computers" in the search bar and open it. Inside AD, right-click on "mydomain.com" on the left and go to "New" and click "Organizational Unit". Name this unit "_EMPLOYEES" then hit "Ok". Repeat this steps for a second organizational unit and call it "_ADMINS".</p>
<p align="center"><img alt="Screenshot 2025-06-26 at 11 40 28 AM" src="https://github.com/user-attachments/assets/c7de6c10-8961-4ada-b070-59c142a59e8a" height="80%" width="80%"/>
</p>
<br />
<p>Now that we have an organizational unit for our admins, we're going to create our first admin account. Right-click "_ADMINS" and go to "New" then click "User". Name the admin "Jane Doe" and her username "jane_admin".</p>
<p align="center"><img alt="Screenshot 2025-06-26 at 11 45 37 AM" src="https://github.com/user-attachments/assets/dff54b9b-96c2-4b43-b90a-f9429cbbaf6b" height="80%" width="80%"/>
</p>
<br />
<p>Click "Next" and for her password choose something you'll easily remember, and for the sake of this tutorial, make sure "User must change password at next login" is unchecked. Click "Next", then "Finish", and now we have our first admin account in our domain.</p>
<br />
<p>This account </p>

























































