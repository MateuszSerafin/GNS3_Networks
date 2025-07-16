# Chapter 10: Terminals, Shells, and Filesystem Troubleshooting
My study guide starts with clarification of what is shell and terminal <br>
Shell: A program that accepts commands and gives them to the operating system to be executed (Bash, ZSH) <br>
Terminal: A terminal is a program that allows end users to interface with the shell (GNOME Terminal, KDE Konsole)

There are multiple shells to chose from, by default you would have either bash or zsh depending on your distribution it could be something entirely different <br>
All of them do the same job, they differ slightly, but it's not like using one or another makes you better or anything <br>
On the other side zsh has cool plugins for coloured directories, autocompletion etc. which is damn cool <br>

## Basic Shell Scripting
Shell scripting is literary just a text file with commands that you would type to terminal but as a file that you can execute <br>

Let's create a basic shell script <br>

```
#!/bin/bash
#A comment
echo Today is $(date +%Y-%m-%d)
```

`#` is used to comment the file, you can put your comments <br>
In case of Shell scripts the first line should be ``#!/bin/(shell with which you want to execute the script with)``. If you do not include this line it will use the default shell of your system (just worth noting) <br>
Before running the script, we need to mark is as executable with ``chmod +x (script.sh)``
```
[root@server ~]# nano test.sh
[root@server ~]# chmod +x test.sh 
[root@server ~]# ./test.sh 
Today is 2025-07-15
[root@server ~]# echo Today is $(date +%Y-%m-%d)
Today is 2025-07-15
[root@server ~]# 
```
As you can see we executed the script and I executed the same line in my terminal yielding the same results <br>

Note: Bash scripts are useful for repeating tasks or more complicated things that you might not want to do manually <br>

Note2: Bash have a PATH <br>
```
[root@server ~]# echo $PATH
/usr/local/sbin:/usr/local/bin:/usr/bin:/usr/bin/site_perl:/usr/bin/vendor_perl:/usr/bin/core_perl
```
We can see there are multiple directories inside a $PATH. What it does is when we type any command, it checks if it exists in those folders and executes them <br>
If you don't have set PATH or you manage to break it. Trying to execute a program will result in an executable not being found (because executables are looked inside $PATH directories) <br>
We can add multiple directories, the directories are split with `:`. And the directives are checked in order and first match is executed <br>

## Conditionals
I will not make it a programing tutorial, so I will just provide examples <br>

A simple script that checks if file exists <br>
```
[root@server ~]# cat test.sh 
if [ -a testfile ]; then
echo "File exists";
else
echo "File doesn't exist"
fi
[root@server ~]# ./test.sh 
File doesn't exist
[root@server ~]# touch testfile
[root@server ~]# ./test.sh 
File exists
[root@server ~]# 
```

## For loops
Iterate over files in directory and print the file if it has letter ``a`` <br>
```
[root@server ~]# cat test.sh 
for file in /bin/*; do
  if [[ $file =~ "a" ]]; then
    echo "$file"
  fi
done
[root@server ~]# ./test.sh 
/bin/aclocal
/bin/aclocal-1.18
/bin/addftinfo
/bin/addgnupghome
...
```

## While loops
Read user input till it's not exactly ``finish`` <br>
```
[root@server ~]# cat test.sh 
read -p "Entry something: " userInpt

while [[ $userInpt != "finish" ]]; do
 read -p "Entry something: " userInpt;
done  

echo "You managed to type finish"
[root@server ~]# ./test.sh 
Entry something: qweqwewqewqwqeq
Entry something: rtuerptuoiuewrptuiewr
Entry something: hello
Entry something: finish
You managed to type finish
[root@server ~]# 
```

## Additional notes to bash scripting
Bash do not have classes, you can create functions <br>
I used bash scripts for basic things maybe up to 50 lines of code <br>
If I need something more advanced I would just use python to do it xd unless it would be significantly faster to do with bash

## Filesystem Troubleshooting
When your system crashes, there is a power outage. There is a chance that your file system might break <br>
You can use fsck, to try repair the filesystem. It might succeed it might not <br>
fsck not only tries to repair the filesystem but also verifies the integrity of it <br>
If fsck finds corrupted files it will place it in ``lost+found`` directory inside root of your filesystem <br>

| Command                 | Description                                                                                                        |
|-------------------------|--------------------------------------------------------------------------------------------------------------------|
| fsck -af (block device) | Automatically repair the file system without asking questions, and force the check even if file system looks clean |
| fsck -n  (block device) | Checks the filesystem does not perform any repairs, outputs information to console                                 |
| fsck (block device)     | Will check if filesystem is dirty will ask questions on what you want to do                                        |

Note: If you have additional packages installed you can do ``fsck.ntfs`` or other filesystem types <br>
With this particular example ``fsck.ntfs`` was unable to recover my ntfs filesystem but when I tried to do with Windows's ``chkdisk`` it recovered it correctly <br>
So it's worth trying different tools if available