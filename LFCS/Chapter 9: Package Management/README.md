# Chapter 9: Package Management
## Common Commands
Depending on your distribution different package manager will be installed, some use apt, some use yum, some use pacman, some use what ever <br>
Covering everything is possible but infeasible, the common trend there is you can do something like `apt install package` and syntax will be only slightly different to other package managers <br>

I think using one table for this whole chapter is sufficient. The only reason I chose apt and pacman is apt is common, pacman is on arch linux, and I am on arch linux 

| APT                                        | PACMAN                            | Description                                                                                                                                      |
|--------------------------------------------|-----------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------|
| apt update                                 | pacman -Sy                        | Update package repositories                                                                                                                      |
| apt install (package)                      | pacman -Sy (package)              | Install package                                                                                                                                  |
| apt upgrade                                | pacman -Syu                       | Update all packages                                                                                                                              |
| apt remove (package)                       | pacman -R (package)               | Remove package                                                                                                                                   |
| apt purge (package)                        | pacman -Rcns (package)            | Remove package with dependencies                                                                                                                 |
| apt show  (package)                        | pacman -Si (package)              | Show information about package                                                                                                                   |
| apt-file search -x "(command)"             | pacman -Fy (command)              | Check what package provides a command (really handy)                                                                                             |
| apt autoremove                             | pacman -Qdtq (pipe) pacman -Rns - | Remove orphaned packages (pacman -Qdtq just shows orphaned packages in a form that is pipable to pacman -Rns which performs the actual deletion) |
| apt install (package.deb)                  | pacman -U (package.tar.gz)        | Install package from local file                                                                                                                  |
| apt list --installed (pipe) grep (package) | pacman -Qi (package)              | Search for package installed on a system                                                                                                         |

Note: While this is not everything It's more than enough 