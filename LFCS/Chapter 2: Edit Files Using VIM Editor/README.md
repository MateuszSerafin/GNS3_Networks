# Chapter 2: Edit Files Using VI/Vim Editor
I always use nano, I use vim rarely mostly when I forget to install nano and I have to configure network interface to download nano lol <br>
It would be good to know at least some shortcuts as I didn't go that much in depth with it <br>

## Just a mindmap really
| Description             | Nano               | Vim                                 |
|-------------------------|--------------------|-------------------------------------|
| Exit without saving     | ctrl+x             | :q!                                 |
| Exit and save           | ctrl+x+y+enter     | :wq                                 |
| Find text               | ctrl+w             | /(find)   + n (for next occurrence) |
| Find and replace all    | ctrl+\+A           | :s/(find)/(replace)/g               |
| Find and replace one    | ctrl+\+y+ctrl+c    | :s/(find)/(replace)/gc              |
| Go with cursor to line X | ctrl+shift+_       | :(line number)                      |
| Cut line                | ctrl+k             | d$                                  |
| Copy line               | alt+6              | yy                                  |
| Copy multiline          | ctrl+6+alt+shift+6 | v+d                                 |
| Paste line              | ctrl+u             | p                                   |
| Wrap lines              | ESC+$              | N/A does that by default            |