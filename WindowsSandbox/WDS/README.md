# WDS
In this example I am going to:
- Capture Windows 10 image with Shotcut and Gimp
- Configure WDS to automatically deploy Windows 10
- Automatically join to domain 

## Lab Setup
![](Lab.png) <br>

## Initial Configuration Of Windows Server
1. Configure ADDS <br>
![](media/InstallationOfDomainController.webp)
2. Configure DHCP + WDS <br>
![](media/DHCPandWDSinstall.webp)
3. Create a secure domain user to join the domain <br>
In unattended.xml we have to specify user that is in domain and will join the computer to the domain. The problem is the password will be in plain text and providing something like domain administrator is unsecure. <br>
Here is a good configuration of this type of user.<br>
![](media/CreationOfDomainJoinUser.webp)

## Preparation of Deployment
1. Create capture Image on WDS <br>
![](media/PreparationForCapturingTheImage.webp)
2. Create Required Unattended.xml <br>
It's important to note that WDS handles passes 1-4 meaning if you specify to skip OOBE which is in pass 7. Nothing actually will happen and user will be greeted with OOBE screen. <br>
To actually do this correctly make sure to split your unattend to two files one for WDS one for the image itself. <br>
Additionally previously I have shown how to create an Unattended file I will provide my configuration there only. <br>
Also make sure to include first reboot, like I did in my configuration as without it the PC will join to domain however you won't be able to log in. <br>
![](media/UnattendedPassesConfig.webp)
3. Prepare the capture image <br>
In my example I will install Shotcut and Gimp before capturing the image <br>
Additionally make sure to put your PASS7.xml to ``C:\Unattend.xml``. There are other location options but those didn't work for me for some reason. <br>
![](media/PreparationOfWindowsCapture.webp) <br>
Make sure to run sysprep! 

## Capturing the image
Capturing the image is pretty easy, you just pxe boot and select the capture image. <br>
However make sure that you have usb or external hard drive plugged in because it won't let you do the capture without other disk <br>
![](media/CapturingTheImage.webp)

## Finishing Configuration
At this point you captured the image, and it will show on WDS. <br>
Make sure to apply your unattended passes1-4 in WDS. <br>
![](media/WDSDeploymentConfig.webp)

## Success
![](media/Success.webp)