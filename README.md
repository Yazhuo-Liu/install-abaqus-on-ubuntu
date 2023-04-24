# Install-ABAQUS-on-Ubuntu
This is a tutorial of installing abaqus2023 on ubuntu 20.04/22.04 and link with ifort for user subroutine.

Note: 
1. The version difference between ifort and abaqus should not be too large, otherwise there will be compatibility problems
2. To successfully follow this tutorial, you need to have root access (sudoers)

## References:
- https://github.com/franaudo/abaqus-ubuntu
- https://www.intel.com/content/www/us/en/docs/oneapi/installation-guide-linux/2023-0/apt.html

## 1. Install prerequisites
The standard Ubuntu release might not have one or more of the following libraries needed by Abaqus:
	-libjpeg
	-libstdc++
	-openmotif
	-libgomp
	-libXm4
	-ksh
	-redhat-lsb-core
	-lsb-release
	-libpng
To install them open a terminal and execute the following command:
```
sudo apt update
sudo apt upgrade
sudo apt autoremove
sudo apt install csh tcsh ksh gcc g++ gfortran libstdc++5 build-essential make libjpeg62 libmotif-dev
```
**please fix all the errors reported in the above processes.**
