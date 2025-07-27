# Chapter 28: Setting up a Network Repository
Imagine if you have a thousand or more machines. Effectively when you are updating your systems you will DOS repository and use precious download bandwidth. <br>
Alternatively you can have local repository and just synchronize the repository and each machine can pull packages from local network <br>

I will not set up the repository however it's really simple <br>
1. You need to find a mirror that supports rsync 
2. rsync the files to local server
3. Pretty much every single package manager (yum, pacman, apt) uses https underneath it hence just start apache server in directory where you rsynced the files <br>
4. Point your machines to your apache instance

That is literary it, I will not do it 