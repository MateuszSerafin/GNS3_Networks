# Tldr
I want to play around with windows automatic installs and want to see what i can do. From what i understand there are 3 options
1. .iso with autounattend.xml
2. Pxe boot with WinPE (still using autounattend.xml)
3. Windows Deployment Services (Didn't use it before)

The disclaimer is i only intend it to work on UEFI

## Creation of basic unattended xml
I followed this [tutorial](https://www.windowscentral.com/how-create-unattended-media-do-automated-installation-windows-10)
<br>
1. Download ADK for your target version i have used [windows 10](https://learn.microsoft.com/en-us/windows-hardware/get-started/adk-install)
2. Using Windows System Image Manager import your target image i have used Education Version
3. Add Components, from what i have seen the component names changed slightly since the tutorial have released but i was still able to find them for my version <br> Pass 1 Windows PE - International-Core-WinPE_neutral, Windows-Setup_neutral <br> Pass 7 oobeSystem - International-Core_neutral, Shell-Setup_neutral <br> I really suggest to use ctrl+f there.
4. Configure the components, there is lots of additional stuff that don't need to be configured, the tutorial i linked provides the bare minimum that will work. It makes a lot of sense when you configure it but there are weird quirks such as configuring the partitions.
<br>After it should look like that![alt text](resources/FirstVersionXML.png)

5. Save it.

## Creating iso with autounattend.xml
Maybe what i did was not the simplest or best option
1. I extracted iso with winrar to folder
2. I added the autounattend.xml generated by Windows System Image Manager
3. I used oscdimg to create bootable iso from folder, the command i provide allows only for boot on UEFI
>oscdimg.exe -m -o -u2 -udfver102 -bootdata:2#p0,e,b(iso folder)\boot\etfsboot.com#pEF,e,b(iso folder)\efi\microsoft\boot\efisys.bin (destination).iso
4. It works
![](resources/FirstVersionInstalling.gif)

## PXE with WinPE
From what i understand there are two options there. Use setup.exe or DISM
### Setting up WinPE
I followed [this document](https://learn.microsoft.com/en-us/windows/deployment/configure-a-pxe-server-to-load-windows-pe) from microsoft. It's pretty much copy and paste.
```
copype.cmd amd64 C:\winpe_amd64
dism.exe /mount-image /imagefile:c:\winpe_amd64\media\sources\boot.wim /index:1 /mountdir:C:\winpe_amd64\mount
copy c:\winpe_amd64\mount\windows\boot\pxe\*.* y:\Boot
copy C:\winpe_amd64\media\boot\boot.sdi y:\Boot
copy C:\winpe_amd64\media\Boot\Fonts y:\Boot\Fonts
bcdedit.exe /createstore c:\BCD
bcdedit.exe /store c:\BCD /create {ramdiskoptions} /d "Ramdisk options"
bcdedit.exe /store c:\BCD /set {ramdiskoptions} ramdisksdidevice boot
bcdedit.exe /store c:\BCD /set {ramdiskoptions} ramdisksdipath \Boot\boot.sdi
```
>bcdedit.exe /store c:\BCD /create /d "winpe boot image" /application osloader

The command above will provide {GUID1} you just replace it

```
bcdedit.exe /store c:\BCD /set {GUID1} device ramdisk=[boot]\Boot\boot.wim,{ramdiskoptions} 
bcdedit.exe /store c:\BCD /set {GUID1} path \windows\system32\winload.exe 
bcdedit.exe /store c:\BCD /set {GUID1} osdevice ramdisk=[boot]\Boot\boot.wim,{ramdiskoptions} 
bcdedit.exe /store c:\BCD /set {GUID1} systemroot \windows
bcdedit.exe /store c:\BCD /set {GUID1} detecthal Yes
bcdedit.exe /store c:\BCD /set {GUID1} winpe Yes
bcdedit.exe /store c:\BCD /create {bootmgr} /d "boot manager"
bcdedit.exe /store c:\BCD /set {bootmgr} timeout 30 
bcdedit.exe /store c:\BCD -displayorder {GUID1} -addlast
copy c:\BCD y:\Boot\BCD
```
The only thing that is important there is that winload.exe is for bios, if you want efi (like in my case) you need to have winload.efi, make sure to change it accordingly, also make sure it's in directory, in my case commands provided by microsoft copied the file. If not just search it in explorer.

>dism /unmount-image /mountdir:c:your_boot_dir\mount
>
Also at the end of this step i used the following command, because in the commands it says to mount it, but it does not mention to unmount. Just a cleanup task

### Setting Actual PXE
Now this is where i encountered some issues.
1. I thought about using WDS's tftp for it, but it looks like it was not intended to be used this way. I might be incorrect with this one but i looked at it and just didn't feel correct.
2. I tried to setup ArchLinux with dnsmasq, I had success previously with pxe booting Arch through it, so I tried WinPE. Upon correctly configuring the PXE it was booting partially.  
![](resources/correctdnsmasq.gif)
<br><br> No matter what I did to it, it just didn't want to play. But the fact that it was doing something got me curios. Upon further investigation i saw that the paths were mixed with Windows forward slashes and unix's backslashes. 
![](resources/othertftpserver.png)
I decided to ditch it.
There are two probable solutions 1. There is weird option in XYZ tftp server to resolve paths correctly 2. It's something to do with WinPE itself like in a setup step or something like that (unlikely). But testing multiple tftp servers just for that when i will ditch this PXE setup after sucessful install seems not worth it.
