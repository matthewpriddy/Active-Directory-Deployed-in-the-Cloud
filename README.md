<p align="center">
<img src="https://i.imgur.com/pU5A58S.png" alt="Microsoft Active Directory Logo"/>
</p>

<h1>Active Directory Deployed in the Cloud (Azure)</h1>
This tutorial outlines the implementation of on-premises Active Directory within Azure Virtual Machines.<br />



<h2>Environments and Technologies Used</h2>

- Desktop PC
- Microsoft Azure (Virtual Machines/Compute)
- Remote Desktop
- Active Directory Domain Services
- PowerShell

<h2>Operating Systems Used </h2>

- Windows Server 2022
- Windows 10 (21H2)

<h2>High-Level Deployment and Configuration Steps</h2>

- Deploy two Virtual Machines (VM) within Microsoft Azure with the following specifications:
  - Domain Controller VM (DCVM) (Windows Server 2022) -- Private IP Address set to STATIC
  - Client VM (Windows 10) -- sharing the same Resource Group and Vnet as the DCVM
- Ensure Connectivity between the Client VM and the DCVM from your Desktop
  - Enable Inbound Rules for "Core Networking Diagnostics" within DCVM's Firewall
- Install Active Directory Domain Services within DCVM (promote DCVM to a Domain Controller immediately following install)
- Create an Admin and Standard User Account in Active Directory (via DCVM)
- Join the Client VM to the Domain
- With the Admin User Account login, setup the Client VM for non-administrative users to gain access
- While creating multiple non-admin Users with the Domain Controller, attempt to log into Client VM as one of those newly created users

<h2>Deployment and Configuration Steps</h2>

We need to begin by creating a Domain Controller VM and a Client VM using Microsoft Azure.

If you would like to learn how to create Virtual Machines using Microsoft Azure, left click the link ["Creating Resource Groups and Deploying Virtual Machines"](https://github.com/matthewpriddy/azure-vm-resource) to learn more.

Lets start with creating a **Resource Group** by using the Azure Search bar at the top of the homepage to search "resource", selecting Resource in the center afterwards.

- - - -
  
</p>
<br />

![image](https://github.com/matthewpriddy/Active-Directory-Deployed-in-the-Cloud/assets/132313534/fd5ea2d1-2561-4589-92c7-deec13f07619)

</p>
<p>
  
- - - -
  
</p>
<br />

![image](https://github.com/matthewpriddy/Active-Directory-Deployed-in-the-Cloud/assets/132313534/0e1a978c-28aa-4192-ab68-c7097550a6ce)


</p>
<p>
  
- - - -
  
</p>
<br />

![image](https://github.com/matthewpriddy/Active-Directory-Deployed-in-the-Cloud/assets/132313534/e7176d1e-5791-4b83-964d-cb5ee94f33ed)

</p>
<p>

Now that we have our **Resource group** created, let's move on to creating the Virtual Machine that will be our Domain Controller. From the Microsoft Home page, using the search bar, search Virtual Machine.

![image](https://github.com/matthewpriddy/Active-Directory-Deployed-in-the-Cloud/assets/132313534/a41a713c-8e1a-4422-a39b-10fb49996a87)



On the create a Virtual Machine page, ensure the following:

► Use created Resource Group [for my example, it is AD-Lab]

► Name the VM as the Domain Controller. In this example, the name chosen was DC1.

► For the Image selection, select - Select Windows Server 2022

► For the Region selection, select: (US) West US 3

► For the Size selection, select: Standard_E2s_v3 - 2vcpus, 16 GiB memory

► Create a Username and password of your choosing (keep it handy, saved or written somewhere safe)

► Click the box near the bottom of the page, select review and create 

![image](https://github.com/matthewpriddy/Active-Directory-Deployed-in-the-Cloud/assets/132313534/715c6b01-5a1c-4a9c-8b8c-1ff07bc65327)

![image](https://github.com/matthewpriddy/Active-Directory-Deployed-in-the-Cloud/assets/132313534/266eae58-65b1-4c50-b9f9-707d2a87a2aa)

It should go through a validation process for a few seconds, then you can select Create to finish creating the DCVM. If it doesn't clear validation for some reason, visit the Networking tab/ page, select Review and Create again to resolve the validation process. 

We don't need to worry about creating a Virtual Network or a sub-net, as it will automatically have been created at this point. 

You will see why that is important shortly, but for now, we will move on to the next step.

- - - -

Let's create a proper Client VM for this tutorial by creating a Virtual Machine with the following:



► Use the same created Resource Group as the DCVM [for my example, it is AD-Lab]

► Name the VM as the Client. In this example, the name chosen was Client1.

► For the Region selection, select - (US) West US 3

► For the Image selection, select - Select Windows 10 Pro

► For the Size selection, select - Standard_E2s_v3 - 2vcpus, 16 GiB memory

► Create a Username and password of your choosing (keep it handy, saved or written somewhere safe).

![Client 1 VM Setup - Part 1](https://github.com/matthewpriddy/Active-Directory-Deployed-in-the-Cloud/assets/132313534/4e0f3af8-b3b2-4d03-966a-1cb16fac92ee)

Click the box near the bottom of the page, select next until you encounter the Networking tab. 

Once there, look at the Virtual Network to ensure the Client VM has the same V-Net as the Domain Controller VM. In this example, it's AD-Lab-vnet.

![Share the same vnet](https://github.com/matthewpriddy/Active-Directory-Deployed-in-the-Cloud/assets/132313534/bece7c8b-836b-49f3-837e-c571d2fb1ebf)

Now select Review and Create, then Create again.

- - - -

We need to adjust the Private IP Address setting on the DCVM. 

From the Azure homepage, search "virtual machines", find the domain controller VM from the results of the search, and left-click the DCVM.

From the Domain Controller VM screen, on the left side, you will notice many selections to choose from. Select Networking.

Now, where it says Network Interface in the center of the screen, click on the blue number next to it to access the Network Interface screen.

![image](https://github.com/matthewpriddy/Active-Directory-Deployed-in-the-Cloud/assets/132313534/244d2bd1-c093-4c8f-b251-017acbd1f83d)

From the Network Interface screen, on the left side, find the "IP Configurations" selection. Select it.

Note the Private IP address status shown on the lower left of the screen, and how it describes the Private IP Address as "dynamic". We need to change that. To the left of the Private IP Address, left-click on where it says "ipconfig1".

![image](https://github.com/matthewpriddy/Active-Directory-Deployed-in-the-Cloud/assets/132313534/711fe19d-bf37-44d8-8cd1-a8c1ffba39be)

Under Private IP Address settings, change the Allocation to Static, then select Save. This is to ensure that the IP address doesn't change.

![image](https://github.com/matthewpriddy/Active-Directory-Deployed-in-the-Cloud/assets/132313534/d9f2158c-9a18-4760-82ea-39277654f79c)

- - - -

We need to ensure there is connectivity between the Client VM and the Domain Controller VM.

To begin with, let's have on hand the login credentials for each VM.

This includes both the Domain Controller VM's and Client VM's: Username, Password, Public IP Address, and the Domain Controller's Private IP Address.

To access the Public IP Address and the Private IP Address, go to Microsoft Azure and access the Virtual Machine's Overview page (search virtual machine at the top, then open both VMs).

![image](https://github.com/matthewpriddy/Active-Directory-Deployed-in-the-Cloud/assets/132313534/c05712cd-7386-4701-bee4-714de9620672)

![image](https://github.com/matthewpriddy/Active-Directory-Deployed-in-the-Cloud/assets/132313534/57947d99-9cc7-42f7-b6d4-e767022bce2d)



We only need to note the Private IP Address for the Domain Controller VM, not to use it as a remote log in, but to help the Client VM ping the Domain Controller VM via the command prompt app.

With the desktop or laptop PC, use Remote Desktop Connection to log in (which you can find by searching "remote" in the Start menu).

Utilizing the credentials you saved when creating the VMs, log in to Client 1 VM.

![image](https://github.com/matthewpriddy/Active-Directory-Deployed-in-the-Cloud/assets/132313534/b5d3c89c-669d-4fce-8813-6afb8bcc5fff)


![image](https://github.com/matthewpriddy/Active-Directory-Deployed-in-the-Cloud/assets/132313534/e3fe0245-7dd6-4637-8b2b-2c128a26060b)

- - - -
From Client 1 VM, open the command prompt via the VM's Start Menu and enter the command ping -t and the Domain Controller VM's private IP address.

Note how there is no traffic between both VMs. 

![image](https://github.com/matthewpriddy/Active-Directory-Deployed-in-the-Cloud/assets/132313534/2ed5a591-4d62-4519-ae7e-4fadf65f891a)

Returning to your desktop or laptop PC, open Remote Desktop Connection to log in to the Domain Controller VM.

From the Start Menu of the Domain Controller VM, search and open: Windows Defender Firewall with Advanced Security.

>Left click Inbound Rules >> Within the Inbound Rules window, sort the Protocol column >>> Within the Protocol column, find ICMPv4 >>>> Hover the mouse cursor over the row that says Core Networking Diagnostics - ICMP Echo Request and Right Click the Mouse to open sub menu>>>>> Left click enable rule.

Under the Enabled column, it should say Yes. Remember to enable rules for both rows.

- - - -

![image](https://github.com/matthewpriddy/Active-Directory-Deployed-in-the-Cloud/assets/132313534/51116ef7-74f6-4318-824a-ba2b4ec04309)


- - - -

Now return to the Client 1 VM and observe the traffic between this VM and the Domain Controller VM. We should have evidence of connectivity.

![image](https://github.com/matthewpriddy/Active-Directory-Deployed-in-the-Cloud/assets/132313534/d4e425ac-efb9-4df6-912f-833d15f57769)

- - - -

It is now time to install Active Directory on the domain controller VM. 

With Server Manager open, click on Add Roles and Features and click Next.

![image](https://github.com/matthewpriddy/Active-Directory-Deployed-in-the-Cloud/assets/132313534/c0644c0d-d3ce-4c35-a459-5ceffe910900)

- - - -

Confirm the private IP address of the domain controller VM.

![image](https://github.com/matthewpriddy/Active-Directory-Deployed-in-the-Cloud/assets/132313534/93f10ce1-7f9a-48c0-801f-03ef8c4ecafe)

- - - -

In the Server Roles tab, click on Active Directory Domain Services. Click Add Features, click Next, then Install.

![image](https://github.com/matthewpriddy/Active-Directory-Deployed-in-the-Cloud/assets/132313534/ad4a1441-68cc-4eaf-ad31-c8830ce76105)

- - - -

![image](https://github.com/matthewpriddy/Active-Directory-Deployed-in-the-Cloud/assets/132313534/259f3dcb-cf12-40a9-9de8-7c931f7edf15)

- - - -

![image](https://github.com/matthewpriddy/Active-Directory-Deployed-in-the-Cloud/assets/132313534/667eb2b9-e6be-4f49-9cdf-91a12f20457b)

- - - -

Next, we have to promote the server into a domain controller. In Server Manager, there is a warning sign in the top right corner under a flag.

This is a good time to save any work on your VM because at the end of this next step, the VM will probably automatically restart. Once that is done, proceed.

Click on that flag and click Promote this server to a domain controller.

![image](https://github.com/matthewpriddy/Active-Directory-Deployed-in-the-Cloud/assets/132313534/c7575b3d-7d17-4027-b19b-008477b69acd)

- - - -

Click on Add a new forest and specify a domain name. In my case, I will use mattpriddy.com.

![image](https://github.com/matthewpriddy/Active-Directory-Deployed-in-the-Cloud/assets/132313534/c7b8d3de-5f82-474c-ba33-f8c8ebab35d4)

- - - -

Specify a password for the domain and click on Next on each screen and Install.

![image](https://github.com/matthewpriddy/Active-Directory-Deployed-in-the-Cloud/assets/132313534/a1b52db7-9c0f-40e3-bc42-fc10b8b51b76)


![image](https://github.com/matthewpriddy/Active-Directory-Deployed-in-the-Cloud/assets/132313534/a8743214-b2b0-40d4-9517-2195474f6532)


![image](https://github.com/matthewpriddy/Active-Directory-Deployed-in-the-Cloud/assets/132313534/520d1613-3000-45d6-9d63-cd966d825fda)

- - - -

End of tutorial.









  



