<p align="center">
<img src="https://i.imgur.com/pU5A58S.png" alt="Microsoft Active Directory Logo"/>
</p>

<h1>On-premises Active Directory Deployed in the Cloud (Azure)</h1>
This tutorial outlines the implementation of on-premises Active Directory within Azure Virtual Machines. Active Directory allows us to manage thousands of user accounts, passwords, and devices in a single space. It also allows us to control access to resources on a large scale. Whenever we hear of people who work at help-desk speak about performing tasks such as reseting passwords, chances are that it was done in Active Directory. In this tutorial, I will show how to set everything up. Enjoy. <br />


<h2>Environments and Technologies Used</h2>

- Microsoft Azure (Virtual Machines/Compute)
- Remote Desktop
- Active Directory Domain Services
- PowerShell

<h2>Operating Systems Used </h2>

- Windows Server 2022
- Windows 10 (21H2)

<h2>High-Level Deployment and Configuration Steps</h2>

- Setup Resources in Azure
- Ensure Connectivity between the client and Domain Controller
- Install Active Directory
- Create an Admin and Normal User Account in AD
- Join Client-1 to your domain (mydomain.com or whatever you wanted to name yours)
- Setup Remote Desktop for non-administrative users on Client-1
- Create a bunch of additional users and attempt to log into client-1 with one of the users

<h2>Deployment and Configuration Steps</h2>

<p>
<img src="https://i.imgur.com/1Cst138.jpg" height="100%" width="100%" alt="Disk Sanitization Steps"/>
</p>
<p>
As I stated before, AD allows us to centrally control resources, user accounts, and devices. But before we can do that, we have to set up and house the AD on a server or what we call a "Domain Controller" (DC-1). So if we can imagine a university where there are thousands of students, we want to make it so that each student can log into any computer (Client) that is "joined" to the AD with their unique login credentials (account name/password). This will allow students to do things such as see the same desktop that is unique to them from any computer that they log in from and utilize all the information stored. In this lab, the focus will mainly be on permissions and user accounts but there are steps we must do before we get to that part. In this section, We will start by setting up our Active Directory. Since I am doing this from home for demonstration purposes, I will be doing this whole process through Virtual Machines in Microsft Azure to simulate a domain controller and a client desktop that will be joined to it. Now to clarify, there are two versions of the active directory. One version is the "on-premise" version which is software that you install on a computer you own. The second version is called Azure Active Directory which is NOT what we are doing here. We will be setting up the on-premise version USING Azure.
</p>
<br />

<p>
<img src="https://i.imgur.com/saZR1Y8.jpg" height="100%" width="100%" alt="Disk Sanitization Steps"/>
<img src="https://i.imgur.com/6GSnCvB.jpg" height="100%" width="100%" alt="Disk Sanitization Steps"/>
</p>
<p>
I am going to start by creating a Virtual Machine in Azure. We are going to be inside of a virtual network which will automatically be created when we are in the VM. When you create a VM in Azure, a bunch of other things get created and one of the things that get created is what is called a NIC or Network Interface Card where IP addresses are assigned to it. We are also going to create a Domain Controller which will also have a NIC. We know that when computers get IP addresses in two ways. One of those is through a DHCP where the home network serving as a DHCP assigns an IP. The same thing happens in Azure where both a DHCP and DNS server is hidden on the Azure server. When we create our DC we will get automatically get an IP from the VP. Usually, when you have some type of server offering services such as in our case with the DC, we don't want the IP address changing or being "Dynamic". So once we create our Domain Controller we will change the settings from Dymaic to Static. We will also be creating our "Client-1" to have Windows 10 and we will be joining it in the DC. We know that whenever we create a VM in Azure, that VM will automatically use the DNS built into Azure. We will be configuring the DNS on our "Client-1" to use the DNS of our Domain Controler instead of the one automatically assigned by Azure. The reason we want to do this is that whenever Active Directory is installed on a server and that server becomes a Domain Controller, a DNS service is installed as well. For the Client to be able to join the domain, it has to use the DNS of the DC as its DNS setting. I will explain this further in the next lab repository that I post after this. 
</p>
<br />


<p>
<img src="https://i.imgur.com/96Kqktg.jpg" height="100%" width="100%" alt="Disk Sanitization Steps"/>
<img src="https://i.imgur.com/daFK83r.jpg" height="100%" width="100%" alt="Disk Sanitization Steps"/>
<img src="https://i.imgur.com/n5xzLGO.jpg" height="100%" width="100%" alt="Disk Sanitization Steps"/>
</p>
<p>
Let us begin by creating a Virtual Machine in Microsoft Azure. We can do this by going to our Azure Portal > Virtual Machines > Create. Azure Virtual Machine.

</p>
<br />

<p>
<img src="https://i.imgur.com/NO4vVXk.jpg" height="100%" width="100%" alt="Disk Sanitization Steps"/>
<img src="https://i.imgur.com/uzpUQ03.jpg" height="100%" width="100%" alt="Disk Sanitization Steps"/>
</p>
<p>
Once we are on our "Create a virtual machine" page we can go ahead and create a new resource group and name it whatever you want. I named mine "AD-Lab". For the image, we want to use Windows Server 2022 as that is going to act as our Domain Controller. We can go ahead and name the VM "DC-1", set security to trusted, and set our region to whatever we want. I used West US 3. Remember, both VMs that we create must be set to the same region otherwise this will not work and you will have to do it all over again. Where it says "size" I selected "standard_E2s_v3 - 2 vcpus, 16 GiB memory" as it has two CPUs just to speed things up. I also created a user named "labuser" and gave it a password which can be anything you want, just don't forget what it is! Go ahead and click Review+Create, let it validate, and click Create. Now we can move on to setting up our Client-1 VM. 
</p>
<br />


<p>
<img src="https://i.imgur.com/jJRYsdN.jpg" height="100%" width="100%" alt="Disk Sanitization Steps"/>
</p>
<p>
Following the same steps as we did for creating our DC-1, I went ahead and set up "Client-1" which will act as our user desktop. Instead of using Windows Server, we are going to set the image to use Windows 10 and remember to place it into the same resource group and region as our DC-1 VM. Once again, set up the same username and password and create. 
</p>
<br />

<p>
<img src="https://i.imgur.com/h73ORUc.jpg" height="100%" width="100%" alt="Disk Sanitization Steps"/>
<img src="https://i.imgur.com/u8dNpDY.jpg" height="100%" width="100%" alt="Disk Sanitization Steps"/>
<img src="https://i.imgur.com/hdhLo0V.jpg" height="100%" width="100%" alt="Disk Sanitization Steps"/>
</p>
<p>
Now that both our domain controller and client are set up we can go ahead and change the private IP of DC-1 from dynamic to static. We can do this by going to DC-1 from our virtual machines page and clicking "Networking" on the left-hand side. We should see something that says "Networking Interface" with the name of our domain controller next to it. Yours might appear different from mine but just look at the above image for reference. We can then click "IP Configurations" on the left and then click on the bottom where it says "Private IP Address" and a new window will open up. Go to where it says "Assignment" and change it from Dynamic to Static. Click save. See the above photo for reference.

</p>
<br />


<p>
<img src="https://i.imgur.com/VLcvgQu.jpg" height="100%" width="100%" alt="Disk Sanitization Steps"/>
<img src="https://i.imgur.com/Xgy9tfG.jpg" height="100%" width="100%" alt="Disk Sanitization Steps"/>
</p>
<p>
Let us log into Client-1 using the remote desktop connection from your actual computer. You can find this by going to your Windows search bar and typing in "Remote Desktop Connection". We can do this by going to Client-1 and copying the Public IP address into Remote Desktop and logging in as the account that we created (labuser).
</p>
<br />


<p>
<img src="https://i.imgur.com/iBLZvmU.jpg" height="100%" width="100%" alt="Disk Sanitization Steps"/>
</p>
<p>
From here we can go ahead and try to ping DC-1's private IP address. To get DC-1's private IP address we can simply just click on DC-1 from our VMs in our Azure portal and go down to where it says "Private IP Address". For me, it's 10.0.0.4 so just go ahead and copy whatever it shows for you. We can then go back into our Client-1 VM on a remote desktop connection and open our command prompt. We can do this by going to our search bar in our start menu and typing "cmd". We can then try to do a perpetual ping of DC-1's private IP by typing "ping -t 10.0.0.4" and pressing enter. We should see that it now continuously says "Request timed out". This is because DC-1's Windows firewall is blocking ICMPv4 traffic. Our next step is going to be enabling it.

</p>
<br />


<p>
<img src="https://i.imgur.com/vC28MS4.jpg" height="100%" width="100%" alt="Disk Sanitization Steps"/>
</p>
<p>
Using the same steps we did to log into Client-1 Remote Desktop Connection, we can do the same for DC-1. Go back to your Azure portal on your actual desktop, go to DC-1, copy the public IP Address, go to the remote desktop connection, and log in. Once inside DC-1, we should see something like the above image.
</p>
<br />


<p>
<img src="https://i.imgur.com/p5XQWr5.jpg" height="100%" width="100%" alt="Disk Sanitization Steps"/>
<img src="https://i.imgur.com/7aYiVyp.jpg" height="100%" width="100%" alt="Disk Sanitization Steps"/>
</p>
<p>
We can go ahead and access the firewall by going to our Windows search bar and either typing "firewall" and selecting "Windows Defender Firewall with Advanced Security on Local Computer" OR typing in "wf.msc" which will take you to the same screen. From there we can go ahead and click on "Inbound Rules" on the left-hand side and then sort the menu by "Protocol". We want to go ahead and find all the ICMPv4 which is what Ping uses and enable the three selections shown in the photo above. We can either right-click each one and press "enable" or just do it from the right-hand side menu.
</p>
<br />


<p>
<img src="https://i.imgur.com/RRtjAVH.jpg" height="100%" width="100%" alt="Disk Sanitization Steps"/>
</p>
<p>
If we go back into Client-1 and try to ping 10.0.0.4 again we will see that it now works and that IVMPv4 traffic is no longer being blocked by the firewall.

</p>
<br />


<p>
<img src="https://i.imgur.com/KHpGlM8.jpg" height="100%" width="100%" alt="Disk Sanitization Steps"/>
</p>
<p>
The next order of business is installing Active Directory. To do this, we need to go into our DC-1 VM and click on "Add Files and Features" on our server manager and run the installer. Make sure you select "Active Directory Domain Services" when asked to select server roles. This step is very important so make sure that you select the correct one! Go on and click next on everything else and let the installer run. 
  
</p>
<br />

<p>
<img src="https://i.imgur.com/I0cSxmV.jpg" height="100%" width="100%" alt="Disk Sanitization Steps"/>
<img src="https://i.imgur.com/GXCEbPM.jpg" height="100%" width="100%" alt="Disk Sanitization Steps"/>
<img src="https://i.imgur.com/nOVE1rR.jpg" height="100%" width="100%" alt="Disk Sanitization Steps"/>  
</p>
<p>
Once the installer has finished we can go ahead and click on the little flag icon on the top of our server manager dashboard. There should be a little yellow icon that appeared. Simply click the flag and select "Promote this server to a domain controller". From there we can select "Add a new forest" and set the root domain name to mydomain.com (you can use whatever name you want). I chose to go with mydomain.com because this is just a demonstration and this is not going to be public. It will ask you to type in a restore mode password which you can ahead and add. Since this is just a tutorial, this part is not important but it is relevant in the real world to save this password. Just keep pressing Next on everything and we will restart when finished. Since we are using VMs for this, it will probably log you out of your VM. Now if you remember, we are logged into this VM using the "labuser" account. If we try to log in the same way again, it will not allow us. This is because DC-1 is now using the FQDN (fully qualified domain name) which is domain-name\user-account. So, in our case, we need to log in as mydomain.com\labuser and use the same password as before.

  
</p>
<br />


<p>
<img src="https://i.imgur.com/nc5WyKf.jpg" height="100%" width="100%" alt="Disk Sanitization Steps"/>  
</p>
<p>
Now that we are about halfway through this tutorial, we can go ahead and create an admin and normal user account in Active Directory. While in our Server Manager dashboard, we can go ahead and click on "Tools" on the top right and select "Active Directory Users and Computers". Here we can see the UI of AD. Here we can see all of our users in the "Users" folder if we click on mydomain.com on the left. We can also see our Domain Controller inside its folder. We are now going to create what is known as "Organizational Units" inside of mydomain.com which can also be thought of as folders but with a lot more use cases for them. But for now, let's just keep it simple. We can right-click mydomain.com on the lefthand side, > new > Organizational Unit. I am going to do this twice and create two separate OUs. One will be called "_EMPLOYEES" and the other will be called "_ADMINS". 

</p>
<br />


<p>
<img src="https://i.imgur.com/ZBXiWYM.jpg" height="100%" width="100%" alt="Disk Sanitization Steps"/>
<img src="https://i.imgur.com/Ts5RKw9.jpg" height="100%" width="100%" alt="Disk Sanitization Steps"/>
</p>
<p>
In the real world, it is never a good idea to make generic account names such as "ADMIN" or "user". In most cases, everything will always be tied to the individual. For this tutorial, we are using "labuser" and we can continue to do so. However, to make things a little more "realistic", I am going to go on and set up an actual dedicated admin account and log out of labuser. We can start by going to "_ADMINS" and then on the right-hand side, we can right-click the blank area and select new > user > and create a Jane Doe account. I named the account "Jane_admin". Click next, set up a password, and you can uncheck the requirement asking to reset the password.
</p>
<br />


<p>
<img src="https://i.imgur.com/oV4Jeha.jpg" height="100%" width="100%" alt="Disk Sanitization Steps"/>
</p>
<p>
Now, even though Jane Doe is in the _ADMINS Organizational Unit does not mean that she has admin privileges yet. We have to go on and set those up for her to use. To do this we can go back into "Active Directory Users and Computers" > _ADMINS > Jane Doe > right click > Properties > Member of > and then we can go to where it says "Enter Object names to select". There we can write "domain" and press search names on the right. A new window should pop up with the option for "Domain Admins". We can go ahead and press OK. Then click apply and closeout. We have now given Jane admin permissions. Now log out of DC-1 and try logging in as mydomain.com\Jan_admin (or whatever you decided to name it).

</p>
<br />



<p>
<img src="https://i.imgur.com/Fa61whj.jpg" height="100%" width="100%" alt="Disk Sanitization Steps"/>
</p>
<p>
We can verify which account we are using by going to our command line and typing in "whoami" and it should display the user account.
  
</p>
<br />



<p>
<img src="https://i.imgur.com/CRAR93v.jpg" height="100%" width="100%" alt="Disk Sanitization Steps"/>
<img src="https://i.imgur.com/bAMybt7.jpg" height="100%" width="100%" alt="Disk Sanitization Steps"/>
</p>
<p>
We now have to join "Client-1" to the Domain Controller (DC-1). We need to do this so that we can use the accounts that are set up on the DC on any computer that is part of its network. From the Azure Portal, we need to set Client-1’s DNS settings to the DC’s Private IP address. We know that the private IP address for DC-1 is 10.0.0.4 so we can go to Client-1 in the Azure portal and click on "Networking" and then click Network Interface. From there we can click on "DNS servers" on the left-hand side and change the setting to "Custom" under DNS Servers to 10.0.0.4 and click save (See above image). We can then go ahead and restart Client-1 from its profile page in Azure. I posted an image of this process above if you have trouble finding it. It will log you out of the Client-1 VM. Just simply sign back in using the original account name you set up when creating the VM at the very beginning. In my case, it was "labuser". 

  
</p>
<br />



<p>
<img src="https://i.imgur.com/y01vK0L.jpg" height="100%" width="100%" alt="Disk Sanitization Steps"/>
</p>
<p>
Let us just see if we completed the previous step of connecting Client-1 to the DNS of DC-1. We can go into our command line and type in "ipconfig /all". From there we should see the updated DNS server IP. 
  
</p>
<br />


<p>
<img src="https://i.imgur.com/36vGyRz.jpg" height="100%" width="100%" alt="Disk Sanitization Steps"/>
</p>
<p>
Now let's join Client-1 to the domain controller (DC-1). We can do this by going to our Windows System Settings and clicking "Rename this PC (advanced)". We can then click change, select domain, and name it mydomain.com (or whatever you named yours before). Press ok. Since we are pointing to our domain controller and our DC-1 knows what mydomain.com is, we can now join mydomain.com. A new window will come up and ask us for a name and password. We can go ahead and enter mydomain.com\jane_admin and the password. We should then get a pop-up saying "Welcome to mydomain.com domain." (it might be hidden behind the other windows that you have open so just minimize them and press OK. It will then ask you to restart. Client-1 is now joined with the domain controller!
  
</p>
<br />


<p>
<img src="https://i.imgur.com/ATLpQtn.jpg" height="100%" width="100%" alt="Disk Sanitization Steps"/>
<img src="https://i.imgur.com/DNans04.jpg" height="100%" width="100%" alt="Disk Sanitization Steps"/>  
</p>
<p>
We will now be able to log in from Client-1 using mydomain.com\Jane_admin. this is something that we were not able to do before because DC-1 and Client-1 were not joined together. Now that they are, we can go ahead and log into Client-1 as mydomain.com\jane_admin. Right now, the only person that can log into mydomain.com is labuser and jane_admin. We want to change this to allow other users to be able to remotely log in as well. We can go back into System settings and click on "Remote Desktop" and click "Select users that can remotely access this PC" under User Accounts. There we will get a blank window with nothing populating it but that is where we can see a list of everyone that is allowed to access the computer. We can then click "Add" and yet another window will open up. Now instead of adding ten thousand names one by one, we can type in "domain users" and click "check names". It will show us a built-in group of Domain Users. We can see that the section got populated and we can go and press OK. Now, everyone who will be added to the list of Domain Users will be able to have access.
  
</p>
<br />

<p>
<img src="https://i.imgur.com/Mx55GkV.jpg" height="100%" width="100%" alt="Disk Sanitization Steps"/>
</p>
<p>
If we minimize the Client-1 VM and go into DC-1, we can see the Domain Users group in Active Directory Users and Computers > mydomain.com >users > domain users > members. There you will see all user accounts that get created. Any account that gets created will be added to that domain users group and will be able to log into client-1. Now we will be able to log into client-1 as a regular non-admin user. Normally you’d want to do this with Group Policy that allows you to change MANY systems at once but I will not be doing that.
</p>
<br />


<p>
<img src="https://i.imgur.com/tpBpY5c.jpg" height="100%" width="100%" alt="Disk Sanitization Steps"/>
<img src="https://i.imgur.com/tpBpY5c.jpg" height="100%" width="100%" alt="Disk Sanitization Steps"/>

</p>
<p>
Let us wrap things up by creating a bunch of additional users and attempting to log into client-1 with one of the users. We should be logged in to DC-1 as jane_admin if we are not already there. We will then go into our Windows search bar and find Power Shell ISE. We want to right-click it and run it as administrator. Powershell is a Windows native scripting language that has several uses that are beyond the scope of this tutorial. For this lab, we will be running a script that allows us to create 10000 users made-up users for demonstration purposes. Don't worry, you will see why in the next steps. The script can be found here (https://github.com/joshmadakor1/AD_PS/blob/master/Generate-Names-Create-Users.ps1). Go ahead and copy the entire script by selecting all and copying (cntrl+c).

</p>
<br />


<p>
<img src="https://i.imgur.com/cOup6mp.jpg" height="100%" width="100%" alt="Disk Sanitization Steps"/>
<img src="https://i.imgur.com/nUAk2uU.jpg" height="100%" width="100%" alt="Disk Sanitization Steps"/>
</p>
<p>
We will be observing how this script will generate 10000 different users all with the same password (Password1). You can see this by reading the scripting language above. We want to go back into Powershell ISE and click on New. Paste the entire script in there (cntrl+v) and run the script (little green button on top that looks like a play button). We will see that the script is going to run and start generating user accounts. We can also see that the accounts will be created in the "_EMPLOYEES" OU within our Active Directory.
  
</p>
<br />

<br />

<p>
<img src="https://i.imgur.com/cuRxxzW.jpg" height="100%" width="100%" alt="Disk Sanitization Steps"/>
<img src="https://i.imgur.com/FUW6nnY.jpg" height="100%" width="100%" alt="Disk Sanitization Steps"/>
</p>
<p>
We can go ahead and press the little green play button on Powershell ISE to run the script and observe what happens. You will see the script creating 10000 users and placing them into the _EMPLOYEES only OU within AD.
 
</p>
<br />


<br />

<p>
<img src="https://i.imgur.com/Mx55GkV.jpg" height="100%" width="100%" alt="Disk Sanitization Steps"/>
</p>
<p>
I selected a random name that was generated. For me, it was "bas.pag". I will now log out of Client-1 and try to log in using this user to test if everything works.
 
</p>
<br />


<br />

<p>
<img src="https://i.imgur.com/5nKJfmM.jpg" height="100%" width="100%" alt="Disk Sanitization Steps"/>
</p>
<p>
If the successful login attempt was not confirmed enough, we can run the command and type in "whoami" and we can see that we are logged in as the user we selected.
 
</p>
<br />


<br />

<p>
<img src="https://i.imgur.com/GQMos5W.jpg" height="100%" width="100%" alt="Disk Sanitization Steps"/>
</p>
<p>
If we go back into DC-1 and right-click any of the names, we can see that we can do all sorts of stuff such as disabling accounts and resetting passwords. No doubt that this is something that help desk professionals will constantly be doing throughout their careers. That is all for now, I hope that this tutorial helped demonstrate how to successfully set up an Active Directory and join the network together. Thanks!
 
</p>
<br />
