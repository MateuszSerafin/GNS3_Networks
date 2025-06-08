# SCCM
In this lab I am going to: <br>
- Install SCCM
- Deploy Windows with SCCM
- Deploy 
  - PyCharm to Engineering Computers
  - LibreOffice to Accounting Computers
  - WinRAR with Phased Deployment

## Lab Setup
![](Lab.png) <br>
Make sure for this lab internet access is configured properly otherwise there are issues client side. Don't ask me how I know

## Configuration of DC0
1. Install ADDS
   - Configure Network Interface
   - Change Computer Name
   - Reboot
   - Install ADDS and DNS
   - Configure ADDS then DNS
   
    ![](media/ADDSInstall.webp)
2. Configure DHCP
   - Install DHCP
   - Commit DHCP
   - Configure Scope (Make sure to configure option 66)
   
   ![](media/DHCPConfiguration.webp)
3.  Create a secure domain user to join the domain <br>
    When we create the task sequence this user will be stored in plain text we do not want to provide administrator there <br>
    Here is a good configuration of this type of user <br>
    ![](../WDS/media/CreationOfDomainJoinUser.webp)
4. Create Users and Organizational Units (Engineering and Accounting for my Lab) <br>
   ![](media/OUCreation.webp)

## Installation of SCCM
1. Initial Configuration
   - Configure Network Interface
   - Change Computer Name
   - Reboot
   - Join To Domain
   
   ![](media/SCCMJoiningToDomain.webp)
2. Install SQL Server 2016 <br>
   - Install Features "Database Engine Services" and "Reporting Services - Native"
   - Additionally, change default user that is running the database to "NT Authority\Network Service"

   ![](media/SQLServerInstallation.webp)
3. Install required Features and Roles
   - Roles
      - WSUS
   - Features
      - .NET Framework 3.5 Features
      - Background Intelligent Transfer Services (BITS)
      - Remote Differential Compression

   ![](media/WSUSandFeatures.webp)
4. Install required external dependencies <br>
    - Microsoft Visual C++ Redistributable
    - Microsoft ODBC Driver 18 for SQL Server
    - ADK 
    - ADK WinPE plugin

   ![](media/ExternalDepedencies.webp)
5. Finally Install SCCM
    - Extract SCCM Installer
    - Extend schema
    - Install

   ![](media/InstallationOfSCCM.webp)

## Deployment of Windows
1. Configure Boundaries <br>
   ![](media/BoundaryConfig.webp)
2. Enable PXE on Distribution Share and import your windows image <br>
   ![](media/PXEAndImportImage.webp)
3. Create a Package containing Unattended.xml <br>
   You need Unattened file in order to skip OOBE <br>

   ![](media/PackageCreation.webp)
4. Create Task Sequence 
   - Specify user that will join the PC to domain
   - Edit task sequence and specify package and file name for unattended xml
   
   ![](media/TaskSequenceCreation.webp)
5. Deploy the task sequence <br>
   ![](media/DeployTaskSequence.webp)

At this point everything is configured. <br>
When you are going to boot up with PXE there will be a prompt, you specify the task sequence, and it will install and join to the domain <br>
![](media/WindowsSuccessInstall.webp)

## Deployment of Software
1. Create User and Device groups 
   - Enable User and System Discovery Method
   - Create Accounting/Engineering groups for Users and PCs

   ![](media/CreationOfGroups.webp)
2. Import applications <br>
   - Create a network share that is accessible to everyone 
   - Put your installers into that share
   - Import the applications
   - Distribute Content 
   
   ![](media/CreateApplications.webp) <br>
   Note: It's important to mention that there are two types of installs. <br>
      - System, will run with different account (machine account) and is privileged. It's intended to install software that require privileges to install and usually would install for all users. 
      - User, will run install as a current user, This is intended for apps where you install just for one user. And does not require additional admin privileges 
   
   All 3 apps I used were installed with System option as they did not have option to install only for current user. <br>
   The same thing cannot be said about "ShotCut" for example. You can install it either for only one user OR install it system-wide. Choosing your install type and context switches is important. 
   <br><br>
   Additionally, .msi installers already have information specified in them and are easiest to deploy <br>
   To deploy .exe files, you need to find context switches which are usually on website or by running something like "installer.exe /help" <br>
   Also you need to specify a method on how SCCM determines if an app is successfully installed, In my example I am checking if Uninstall.exe exists in a directory. <br>
   This is not a perfect approach, but it works. From my previous experience you need to look for apps that are installed as a User and can do auto-update which could result in app being updated match detection being failed, and it results in being reinstalled to old version. (In a loop) <br>
3. Normal Deployment <br>
   ![](media/DeployNormal.webp) <br>
   Note: It's good to mention that deploying to Users means that the User will be able to download this particular app on any PC. <br>
   Where deploying to computer means any User will be able to download app on this particular computer. <br>
   In my case I deployed to computers only. 
   
   Additionally, deploying as Available means that a User will be able to download the app. If it's deployed as Required SCCM will install it to a computer without asking user. (User cannot remove it)
   
4. Phased Deployment <br>
   Phased Deployment is really similar to regular deployment the difference being it's just split into two deployments. <br>
   Good example would be you have test machines, and you first deploy it there, once it successfully deployed it will automatically deploy to second group which could be all of your systems. <br>
   In my case I phase deployed WinRAR to EngineeringPC's and then to AccountingPC's <br>

   ![](media/PhasedDeploymentWinRAR.webp)

## Testing of App Deployment
- PyCharm
   ![](media/EngineeringSuccess.webp)
- LibreOffice
  ![](media/SuccessAccounting.webp)
- WinRAR <br>
   Note: Phased Deployment is slow. I left it for 1 hour and it installed. <br>
   You can check if phased deployment correctly deployed on SCCM <br>
   ![](media/PhasedDeploymentPhase1.png) <br>
   ![](media/PhasedDeploymentPhase2.png) <br>
   ![](media/FinalFinal.png)