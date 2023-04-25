# Install-ABAQUS-on-Ubuntu
This is a tutorial of installing ABAQUS2023 and ifort2021 on ubuntu 22.04 and link them for user subroutine. This guide should also work for Abaqus 2021 (changing accordingly file names and paths) and Ubuntu 18.04 till 22.04, although I haven't tested it.

Note: 
1. The version difference between ifort and abaqus should not be too large, otherwise there will be compatibility problems
2. To successfully follow this tutorial, you need to have root access (sudoers)

## References:
- https://github.com/franaudo/abaqus-ubuntu
- https://www.intel.com/content/www/us/en/docs/oneapi/installation-guide-linux/2023-0/apt.html
- https://github.com/Yazhuo-Liu/install-abaqus-on-ubuntu/blob/main/SIMULIA_Established_Products_Installation_Guide.pdf

## 1. Install prerequisites
The standard Ubuntu release might not have one or more of the following libraries needed by Abaqus:

- libjpeg
- libstdc++
- openmotif
- libgomp
- libXm4
- ksh
- redhat-lsb-core
- lsb-release
- libpng

To install them open a terminal and execute the following command:
```
sudo apt update
sudo apt upgrade
sudo apt autoremove
sudo apt install csh tcsh ksh gcc g++ gfortran libstdc++5 build-essential make libjpeg62 libmotif-dev
```
**please fix all the errors reported in the above processes.**

> According to _franaudo_, if there are errors try to install the ones that failed, using the synaptic package manager. To install it, run:
> ```
> sudo apt-get update
> sudo apt-get install synaptic
> ```
## 2. Install ifort
More detailed information, please visit: https://www.intel.com/content/www/us/en/docs/oneapi/installation-guide-linux/2023-0/apt.html
**Download the key to system keyring:**
```
wget -O- https://apt.repos.intel.com/intel-gpg-keys/GPG-PUB-KEY-INTEL-SW-PRODUCTS.PUB \
| gpg --dearmor | sudo tee /usr/share/keyrings/oneapi-archive-keyring.gpg > /dev/null
```
**Add signed entry to apt sources and configure the APT client to use Intel repository:**
```
echo "deb [signed-by=/usr/share/keyrings/oneapi-archive-keyring.gpg] https://apt.repos.intel.com/oneapi all main" | sudo tee /etc/apt/sources.list.d/oneAPI.list
```
**Update the package lists:**
```
sudo apt update
```
**Install the ifort package:**
```
sudo apt-get install intel-oneapi-ifort
```
If you verify the installation now by running `ifort -v`, you will find there is no command names `ifort`. That is because you haven't initialized the package. To initializing ifort, you need to run
```
source /opt/intel/oneapi/setvars.sh
```
And then you can verify the installation
```
ifort -v
```
**Add the initialization step into `.bashrc`**

Add `source /opt/intel/oneapi/setvars.sh` to your `.bashrc` file through:
```
nano ~/.bashrc
```
If you want to initialize ifort for all user, adding `source /opt/intel/oneapi/setvars.sh` to `profile` file through
```
sudo nano /etc/profile
```
Then reconnect or restart the terminal.
Or source the `.bashrc` and `profile` file
```
source ~/.bashrc
source /etc/profile
```
Now you finished the installation of ifort
## 3. Install ABAQUS Products
Note that ABAQUS only officially supports the CentOS and RedHat system, and trying to install Abaqus on Ubuntu will result in an error. In order to skip system detection, we need to modify all the `linux.sh` files in the Abaqus installation folders into the following contant:

**Note the changes:** 
- the release version was forced to be "CentOS"
- disable the prerequisites checking.

```
  DSY_LIBPATH_VARNAME=LD_LIBRARY_PATH

  which lsb_release
  if [[ $? -ne 0 ]] ; then
    echo "lsb_release is not found: check in the PDIR the list of installed packages for servers validation."
    exit 12
  fi

  DSY_OS_Release="CentOS" # Override system information
  echo "DSY_OS_Release=\""${DSY_OS_Release}"\""
  export DSY_OS_Release=${DSY_OS_Release}
  export DSY_Skip_CheckPrereq=1 # Cancel the prerequisite check

  if [[ -n ${DSY_Force_OS} ]]; then
    DSY_OS=${DSY_Force_OS}
    echo "DSY_Force_OS=\""${DSY_Force_OS}"\", use it for DSY_OS"
    return
  fi

  case ${DSY_OS_Release} in
      "RedHatEnterpriseServer"|"RedHatEnterpriseClient"|"RedHatEnterpriseWorkstation"|"CentOS")
          DSY_OS=linux_a64;;
      "SUSELINUX"|"SUSE")
          DSY_OS=linux_a64;;
      *)
          echo "Unknown linux release \""${DSY_OS_Release}"\""
          echo "exit 8"
          exit 8;;
  esac
```
