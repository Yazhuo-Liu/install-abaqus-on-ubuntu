# Install-ABAQUS-on-Ubuntu
This is a tutorial of installing ABAQUS2023 and ifort2021 on ubuntu 22.04 and link them for user subroutine. This guide should also work for Abaqus 2021 (changing accordingly file names and paths) and Ubuntu 18.04 till 22.04, although I haven't tested it.

------------------------------
Update Nov. 11, 2024
- After Intel updated OneAPI, `ifort` no longer exists, and it is replaced by `ifx`. Although `ifx` is fully compatible with the previous `ifort`, it is necessary to create a symbolic link at the end to let `ifort` point to `ifx`, since the default fortran compiler name of abaqus is ifort.
- This tutorial still works well, just change all `ifort` to `ifx`.
- As a service provider, **Intel's behavior is rude, brutal, and unreasonable**. Since `ifx` is fully compatible with `ifort`, there is no need to change the name of the compiler. Intel's behavior is irresponsible to all users, and all users need to bear the cost of the name change.
-------------------------------

Note: 
1. The version difference between ifort and abaqus should not be too large, otherwise there will be compatibility problems
2. To successfully follow this tutorial, you need to have root access (sudoers)

## References:
- https://github.com/franaudo/abaqus-ubuntu
- https://www.intel.com/content/www/us/en/docs/oneapi/installation-guide-linux/2023-0/apt.html
- https://gatech.service-now.com/home?id=kb_article_view&sysparm_article=KB0040327
- https://github.com/Yazhuo-Liu/install-abaqus-on-ubuntu/blob/main/SIMULIA_Established_Products_Installation_Guide.pdf
- https://github.com/Yazhuo-Liu/install-abaqus-on-ubuntu/blob/main/SIMULIA_Installation_Instructions.pdf

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
```bash
sudo apt update
sudo apt upgrade
sudo apt autoremove
sudo apt install csh tcsh ksh gcc g++ gfortran libstdc++5 build-essential make libjpeg62 libmotif-dev
```
**please fix all the errors reported in the above processes.**

> According to _franaudo_, if there are errors try to install the ones that failed, using the synaptic package manager. To install it, run:
> ```bash
> sudo apt-get update
> sudo apt-get install synaptic
> ```

## 2.0 Install a specific version of ifort only to avoid various problems caused by Intel's tricks
The installation package of ifort version 2024.2 can be downloaded [here](/l_fortran-compiler_p_2024.2.1.80.sh)

Just run
```bash
chmod +x l_fortran-compiler_p_2024.2.1.80.sh
sudo ./l_fortran-compiler_p_2024.2.1.80.sh
```
You will finish the installation of ifort.

After installation, add `-diag-disable=10448` to `ifort.cfg` file to turn off Intel's warning that ifort is no longer supported. The default path for this file is `/opt/intel/oneapi/compiler/2024.2/bin/ifort.cfg`

## 2. Install the newest version of ifort 
More detailed information, please visit: https://www.intel.com/content/www/us/en/docs/oneapi/installation-guide-linux/2023-0/apt.html

**Download the key to system keyring:**
```bash
wget -O- https://apt.repos.intel.com/intel-gpg-keys/GPG-PUB-KEY-INTEL-SW-PRODUCTS.PUB \
| gpg --dearmor | sudo tee /usr/share/keyrings/oneapi-archive-keyring.gpg > /dev/null
```
**Add signed entry to apt sources and configure the APT client to use Intel repository:**
```bash
echo "deb [signed-by=/usr/share/keyrings/oneapi-archive-keyring.gpg] https://apt.repos.intel.com/oneapi all main" | sudo tee /etc/apt/sources.list.d/oneAPI.list
```
**Update the package lists:**
```bash
sudo apt update
```
**Install the ifort package:**
```bash
sudo apt-get install intel-oneapi-ifort
```
If you verify the installation now by running `ifort -v`, you will find there is no command names `ifort`. That is because you haven't initialized the package. To initializing ifort, you need to run
```bash
source /opt/intel/oneapi/setvars.sh
```
And then you can verify the installation
```bash
ifort -v
```
**Add the initialization step into `.bashrc`**

Add `source /opt/intel/oneapi/setvars.sh` to your `.bashrc` file through:
```bash
nano ~/.bashrc
```
If you want to initialize ifort for all user, adding `source /opt/intel/oneapi/setvars.sh` to `profile` file through
```bash
sudo nano /etc/profile
```
Then reconnect or restart the terminal.
Or source the `.bashrc` and `profile` file
```bash
source ~/.bashrc
source /etc/profile
```
Now you finished the installation of ifort
## 3. Install ABAQUS Products
### Bypass the system check
Note that ABAQUS only officially supports the CentOS and RedHat system, and trying to install Abaqus on Ubuntu will result in an error. In order to skip system detection, we need to modify all the `linux.sh` files in the Abaqus installation folders into the following contant:

**Note the changes:** 
- Force the release version of LinuxOS to be "CentOS"
- Disable the prerequisites checking.

``` bash
  DSY_LIBPATH_VARNAME=LD_LIBRARY_PATH

  which lsb_release
  if [[ $? -ne 0 ]] ; then
    echo "lsb_release is not found: check in the PDIR the list of installed packages for servers validation."
    exit 12
  fi

  DSY_OS_Release="CentOS" # Override system release
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
### Run Setup
The setup of Abaqus requires a `bash` instead of a `dash` shell, whereas the latter is the default in Ubuntu. We need to temporaryly symlink the shell command to `bash`.
```bash
sudo ln -sf bash /bin/sh
```
Then we can continue the installation.
```bash
cd <download_dir>/<media_name>/1
sudo ./StartGUI.sh
```
follow the guidence provided by Georgia Tech IT Services & Support (https://github.com/Yazhuo-Liu/install-abaqus-on-ubuntu/blob/main/SIMULIA_Installation_Instructions.pdf) and use standard locations for everything. 
**When it asks for the license server, select `Skip licensing configuration` (otherwise it will cause an error).**

The Abaqus will be installed at
```
/usr/SIMULIA/EstProducts/2023
/var/DassaultSystems/SIMULIA/Commands
/var/DassaultSystems/SIMULIA/CAE/plugins/2023
```
Now the installation is complete, and we need to revert the standard shell back to `dash`:
```bash
sudo ln -sf dash /bin/sh
```
## 4. Configure Abaqus for a FLEXnet License Server
Edit the following enviroment customization file in the Abaqus installation.
``` bash
sudo nano /usr/SIMULIA/EstProducts/2023/linux_a64/SMA/site/custom_v6.env
```
In the `custom_v6.env` file, add the following parameters:
```
license_server_type=FLEXNET
abaquslm_license_file="port@license_server_hostname"
```

## 5. Make ABAQUS command available for all users
Creating the symlink in `/usr/bin` dir to make everyone access to the software.
```bash
sudo ln -s /var/DassaultSystems/SIMULIA/Commands/abq2023 /usr/bin/abq2023
```

## 6. If the Intel OneAPI version you installed does not include ifx, and your abaqus prompts that you cannot find ifort, please create a symbolic link
Find the path to `ifx`:
```bash
which ifx
```
copy the path
```bash
sudo ln -s /path/to/ifx /usr/local/bin/ifort
```
!!!!! After doing this symbolic link, there is no need to run `source /opt/intel/oneapi/setvars.sh` in your `.bashrc`

## 7. Check everything
Since we already initialize ifort in our `.bashrc`, the ifort will be automatically linked to ABAQUS.

Now we can start ABAQUS CAE in the terminal. If there is any warning regarding OpenGL in the terminal during start, simply run ABAQUS with `-mesa` parameter:
```bash
abaqus2023 cae -mesa
```
