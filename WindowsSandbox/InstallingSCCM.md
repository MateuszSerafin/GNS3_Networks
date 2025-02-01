# InstallingSCCM
Sometime ago I wanted to mess around with SCCM but when i was searching for how to install it i was seeing 1 hour videos on YouTube or other unnecessarily long tutorials/docs. It greatly discouraged me but today was the day. And it was absolutely horrendously easy.

[This is a tutorial](https://fb.watch/xqdmbNFMl2/) i followed (at least partially)
<br> This is mostly for me, it will be a reminder for future 

1. I installed Windows Server 2025, I also installed Domain Services, DHCP, DNS configured everything for my network.
2. I downloaded SQL server 2016 from microsoft website, created iso
<br> ![](InstallingSCCM/SQLMakingIso.png)
3. Run the installer, click next pretty much for everything aside for 2 things
    - Features, i think "Database Engine Services" is the minimum, but i also used "Reporting Services - Native"
   <br> ![](InstallingSCCM/SQLFeatures.png)
    - For Account that will be running the SQL server i chose "Network Service", it's a account that already exist, (Upon previous testing SCCM installer complained) you can define your own user but it can't be left to default (even the sql server would work)
4. Download ADK and Windows PE
<br> ![](InstallingSCCM/WinPE.png)
Again "Deployment Tools" and "User State Migration Tool (USMT)" is only required nothing else
<br> ![](InstallingSCCM/ADK.png)
5. Additional Roles/Features
    - Install Role "Windows Server Update Services" and Configure
    - Install Feature "Background Intelligent Transfer Service (BITS)" and "Remote Differential Compression"
    Make sure you select SQL Connectivity for WSUS otherwise it's really simple
    <br> ![](InstallingSCCM/SelectCorrectly.png)
6. Download SCCM, extract it, run "extadsch.exe"
![](InstallingSCCM/extadsch.png)
7. (This is copied from the tutorial I linked above)
Open "ADSI edit", connect to your domain controller.
Now, right click the systems container->new->object->container->click next->name it "System Management"->click next. Now we have our systems management container that we'll use to grant security to that SCCM can control and manage the whole container.
<br> ![](InstallingSCCM/stolen1.png)
<br> ![](InstallingSCCM/stolen2.png)
<br> ![](InstallingSCCM/stolen3.png)
From what I have seen the tutorial author named machine "SCCM", I thought this is user but it was not, just make sure to replace that with your current machine name
8. Mostly prerequisites are installed but when I tried it, it died with those errors
   - Net-Framework 3.5 not installed
   - ODBC Driver 18 not installed
   - Visual C++ Redistributable 2017 not installed 
   ![](InstallingSCCM/onlyOne.png)
   Was really easy to solve,
9. Success
<br> ![](InstallingSCCM/Success.png)
I was surprised that it took 6,5 hours, i searched for it, and it wasn't supposed to take that long, I was worried that it was unsuccessful but after checking and considering that it was a VM and my HDD suck and it probably copies lot's of small files started to seem reasonable (Yea need SSD for those vms)
<br> ![](InstallingSCCM/Working.png)

## Test Deploying
1. Create a Boundary
<br> ![](InstallingSCCM/CreatingBoundary.png)
<br> ![](InstallingSCCM/CreatingGroup.png)
<br> ![](InstallingSCCM/CreatingGroup1.png)
2. Enable Discovery of Systems/Users
<br> ![](InstallingSCCM/SystemsDiscovery.png)
<br> ![](InstallingSCCM/DiscoveryUsers.png)
3. Enable Client-Push 
<br> ![](InstallingSCCM/EnablePush.png)
Any Account in Domain Admins is good enough.
<br> ![](InstallingSCCM/AddAccount.png)
4. Adjust Firewall, I created a GPO that deploys settings recommended by microsoft https://learn.microsoft.com/en-us/mem/configmgr/core/clients/deploy/windows-firewall-and-port-settings-for-clients
<br> ![](InstallingSCCM/AdjustingFirewall.png)
5. Because I enabled discovery i am able to see my PC0, I install Client now
<br> ![](InstallingSCCM/InstallingClient.png)
<br> ![](InstallingSCCM/PushingToClient.png)
It's important to mention that "Internet" connectivity is required for client to install successfully. Local network is not good enough. From checking logs it installed some dependencies such as "C++ Redistributable"
6. You can check if it successfully installed in control panel, if you see "Configuration Manager" then it's ALL good 
<br> ![](InstallingSCCM/Installed.png)
7. Create a share that will house your applications
<br> ![](InstallingSCCM/Share.png)
8. Under Software Library and Applications, Create New and put your app on share. Because i used .msi it was really simple and i clicked only next
<br> ![](InstallingSCCM/Addingmsi.png)
9. Distribute Content
<br> ![](InstallingSCCM/Distributing.png)
10. Deploy Content, the only thing i selected is to all users, and it's purpose to be required. 
<br> ![](InstallingSCCM/Deploying.png)
<br> ![](InstallingSCCM/Deploying2.png)
<br> Now from Client side after some time i received a pop-up
<br> ![](InstallingSCCM/ClientSide.png)
<br> ![](InstallingSCCM/ClientSide2.png)
Upon checking i confirmed 7-zip was installed. Now I know I installed it correctly and can continue exploring SCCM.