<p align="center">
<img src="https://i.imgur.com/pU5A58S.png" alt="Microsoft Active Directory Logo"/>
</p>

<h1>Active Directory Deployed in the Cloud (Azure)</h1>
This tutorial outlines the implementation of on-premises Active Directory within Azure Virtual Machines.<br />



<h2>Environments and Technologies Used</h2>

- Desktop PC or MAC
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

(https://github.com/matthewpriddy/azure-vm-resource/blob/main/Create%20Resource%202nd.png?raw=true)
</p>
<p>
  
- - - -
  
</p>
<br />

![test](https://github.com/matthewpriddy/azure-vm-resource/blob/main/Create%20Resource%203rd.png?raw=true)
</p>
<p>
  
- - - -
  
</p>
<br />

![test](https://github.com/matthewpriddy/azure-vm-resource/blob/main/Create%20Resource%204th.png?raw=true)
</p>
<p>

Now that we have our **Resource group** created, let's move on to creating the Virtual Machine that will be our Domain Controller. From the Microsoft Home page, using the search bar, search Virtual Machine.

![test](https://github.com/matthewpriddy/azure-vm-resource/blob/main/Create%20Virtual%20Machine%201st.png?raw=true)



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

You will see why thats important shortly, but for now, we will move on to the next step.

- - - -

Lets create a proper Client VM for this tutorial:

![Client 1 VM Setup - Part 1](https://github.com/matthewpriddy/Active-Directory-Deployed-in-the-Cloud/assets/132313534/4e0f3af8-b3b2-4d03-966a-1cb16fac92ee)

On the create a Virtual Machine page, ensure the following:

► Use the same created Resource Group as the DCVM [for my example, it is AD-Lab]

► Name the VM as the Client. In this example, the name chosen was Client1.

► For the Region selection, select - (US) West US 3

► For the Image selection, select - Select Windows 10 Pro

► For the Size selection, select - Standard_E2s_v3 - 2vcpus, 16 GiB memory

► Create a Username and password of your choosing (keep it handy, saved or written somewhere safe).

Click the box near the bottom of the page, select next until you encounter the Networking tab. 

Once there, look at the Virtual Network to ensure the Client VM has the same V-Net as the Domain Controller VM. In this example, it's AD-Lab-vnet.

![Share the same vnet](https://github.com/matthewpriddy/Active-Directory-Deployed-in-the-Cloud/assets/132313534/bece7c8b-836b-49f3-837e-c571d2fb1ebf)

Now select Review and Create, then Create again.

- - - -

We need to adjust the Private IP Address setting on the DCVM. 

From the Azure hompage, search "virtual machines", find the domain controller VM from the results of the search, and left click the DCVM.

From the DCVM screen, on the left side, you will notice many selections to choose from. Select Networking.

Now, where it says Network Interface in the center of the screen, click on the blue number next to it to access the Network Interface screen.

![image](https://github.com/matthewpriddy/Active-Directory-Deployed-in-the-Cloud/assets/132313534/244d2bd1-c093-4c8f-b251-017acbd1f83d)

From the Network Interface screen, on the left side, find the "IP Configurations" selection. Select it.

Note the Private IP address status shown on the lower left of the screen, how it describes the Private IP Address as "dynamic". We need to change that. To the left of the Private IP Address, left click on where it says "ipconfig1".

![image](https://github.com/matthewpriddy/Active-Directory-Deployed-in-the-Cloud/assets/132313534/711fe19d-bf37-44d8-8cd1-a8c1ffba39be)

Under Private IP Address settings, change the Allocation to Static, then select save. This is to ensure that the IP address doesn't change.

![image](https://github.com/matthewpriddy/Active-Directory-Deployed-in-the-Cloud/assets/132313534/d9f2158c-9a18-4760-82ea-39277654f79c)

- - - -










  



Now that we have deployed the Windows 10 **virtual machine**, lets move on to deploy the Windows Server 2022 **virtual machine**. To do so, lets select Create another **VM**. Repeat the initial deployment steps as for the previously deployed **virtual machine** until you reach the screen under the Basics tab. We will name this **VM** "VM2".
  
- - - -

</p>
<br />

![test](https://github.com/matthewpriddy/azure-vm-resource/blob/main/Create%20Virtual%20Machine%208th.png?raw=true)

- - - -

</p>
<br />

![test](https://github.com/matthewpriddy/azure-vm-resource/blob/main/Create%20Virtual%20Machine%209th.png?raw=true)

- - - -

For the last **VM** which we will call "VM3", we are deploying a Linux **virtual machine**. Repeat the initial deployment steps as for the previously deployed **virtual machine** until you reach the screen under the Basics tab.

- - - -

</p>
<br />

![test](https://github.com/matthewpriddy/azure-vm-resource/blob/main/Create%20Virtual%20Machine%2010th.png?raw=true)

- - - -

</p>
<br />

![test](https://github.com/matthewpriddy/azure-vm-resource/blob/main/Create%20Virtual%20Machine%2011th.png?raw=true)

- - - -

</p>
<br />

![test](https://github.com/matthewpriddy/azure-vm-resource/blob/main/Create%20Virtual%20Machine%209th.png?raw=true)

- - - -

</p>
<br />

![test](https://github.com/matthewpriddy/azure-vm-resource/blob/main/Create%20Virtual%20Machine%205th.png?raw=true)

- - - -

From the search bar at the top of the Azure webpage, search virtual machine. We have deployed three **virtual machines**.

- - - -

</p>
<br />

![test](https://github.com/matthewpriddy/azure-vm-resource/blob/main/search%20bar.png?raw=true)

- - - -

So all that's left for us to do now to wrap up this tutorial, is to remote in to one of these machines from our local PC desktop. Click on "VM2" from the **Virtual machines** screen to copy the Public IP Address of "VM2". 

- - - -

</p>
<br />

![test](https://github.com/matthewpriddy/azure-vm-resource/blob/main/Create%20Virtual%20Machine%2012th.png?raw=true)

- - - -

From the desktop, utilize the search bar of the Start Menu and find "Remote Desktop Connection".

- - - -

</p>
<br />

![test](https://github.com/matthewpriddy/azure-vm-resource/blob/main/remote%20in.png?raw=true)

- - - -

Enter "VM2" as the Username and the password you created for this machine (disregard the VM1 reference in the example pic below). It will take a few moments to boot like a desktop.

- - - -

</p>
<br />

![test](https://github.com/matthewpriddy/azure-vm-resource/blob/main/sign%20in%20to%20vm1.PNG?raw=true)

- - - -

</p>
<br />

![test](https://github.com/matthewpriddy/azure-vm-resource/blob/main/sign%20in%20to%20vm1%20part%202.PNG?raw=true)
