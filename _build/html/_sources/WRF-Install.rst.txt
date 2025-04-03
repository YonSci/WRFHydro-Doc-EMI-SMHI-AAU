WRF Model Installation
=============================================

This document provides a detailed and indepth explanation of a Bash script designed to install the WRF (Weather Research and Forecasting) model, version 4.6.0, along with its prerequisites. The script supports different configurations `WRF-ARW`, `WRF-Chem`, and `WRF-Hydro`  based on a command-line argument. It automates the installation process, including package management, library compilation, and model configuration, for 64-bit Debian-based operating systems (e.g., Ubuntu, Linux Mint).

.. contents:: Table of Contents
   :depth: 3

Prerequisites
-------------

1. **System Requirements**:
   - 64-bit Linux distribution with apt package manager (Tested on Ubuntu 22.04)
   - Minimum 52 GB disk space
   - Non-root user with sudo privileges
   - 64-bit architecture (x86_64)
   - Apt package manager availability


Script Overview
---------------

The script simplifies the installation of WRF 4.6.0 by handling directory setup, dependency installation, and compilation. It is intended for users with sufficient permissions to install packages and assumes a Debian-based system with `apt` package management. The installation can take several hours and requires approximately 52 GB of storage.

Shebang Line
------------

.. code-block:: bash

   #!/bin/bash

**Explanation**: This line specifies that the script should be executed using the Bash shell, ensuring compatibility with Bash syntax and commands.


Set WRF Version and Type
------------------------

The script begins by defining the WRF version and the type of model to be installed. The user can specify the type of WRF model (ARW, Chem, or Hydro) as a command-line argument. If no argument is provided, it defaults to WRF-ARW. The version of the model is set to 4.6.0. 


.. code-block:: bash

   WRFversion="4.6.0"
   type="ARW"
   if [ -n "$1" ]; then
       if [ "$1" = "-chem" ]; then
           type="Chem"
       elif [ "$1" = "-arw" ]; then
           type="ARW"
       elif [ "$1" = "-hydro" ]; then
           type="Hydro"
       else
           echo "Unrecognized option, please run as"
           echo "For WRF-ARW \"bash WRF${WRFversion}_Install.bash -arw\""
           echo "For WRF-Chem \"bash WRF${WRFversion}_Install.bash -chem\""
           echo "For WRF-Hydro \"bash WRF${WRFversion}_Install.bash -hydro\""
           exit
       fi
   fi
   echo "Welcome! This Script will install the WRF${WRFversion}-${type}"
   echo "Installation may take several hours and it takes 52 GB storage. Be sure that you have enough time and storage."

**Explanation**

  - `WRFversion="4.6.0"`: Defines the WRF version to be installed.
  - `type="ARW"`: Sets the default configuration to WRF-ARW (Atmospheric Research Weather).
  - The `if [ -n "$1" ]` block checks for a command-line argument:
    - `-chem`: Sets `type` to "Chem" for WRF-Chem (includes chemistry modules).
    - `-arw`: Keeps `type` as "ARW".
    - `-hydro`: Sets `type` to "Hydro" for WRF-Hydro (hydrological modeling).
    - If the argument is unrecognized, it displays usage instructions and exits.
  - Prints a welcome message with the version and type, and warns about the time and storage requirements.


System Checks
-------------

The script performs system checks to ensure compatibility.

.. code-block:: bash

   if [ "$EUID" -eq 0 ]
     then echo "Running this script as root or sudo, is not suggested"
     exit
   fi

**Explanation**: 
  - `$EUID` checks the effective user ID. If it’s 0 (root), the script warns that running as root or with `sudo` is not recommended and exits, encouraging safer execution as a regular user.


.. code-block:: bash

   osbit=$(uname -m)
   if [ "$osbit" = "x86_64" ]; then
       echo "64 bit operating system is used"
   else
       echo "Sorry! This script was written for 64 bit operating systems."
       exit
   fi

- **Explanation**:
  - `uname -m` retrieves the system architecture, stored in `osbit`.
  - Confirms it’s `x86_64` (64-bit); otherwise, it exits with an error, as the script is designed only for 64-bit systems.


.. code-block:: bash

   packagemanagement=$(which apt)
   if [ -n "$packagemanagement" ]; then
       echo "Operating system uses apt packagemanagement"
   else
       echo "Sorry! This script is written for the operating systems which uses apt packagemanagement. Please try this script with debian based operating systems, such as, Ubuntu, Linux Mint, Debian, Pardus etc."
       exit
   fi

**Explanation**:
  - `which apt` checks for the `apt` package manager, common in Debian-based systems.
  - If found, confirms compatibility; if not, exits with a message specifying supported systems (e.g., Ubuntu, Debian).


.. code-block:: bash

   local_language=$(locale | grep LANG | grep tr_TR)
   if [ -n "$local_language" ]; then
     echo "Merhaba, WRF modelinin kodundaki hatadan dolayı, WRF kurulumu işletim sistemi dili Türkçe olduğunda, Türkçedeki i ve ı harflerinin farklı olması sebebiyle hata vermektedir. Lütfen işletim sisteminizin dilini başka bir dile çevirip yeniden çalıştırınız. Kurulum bittikten sonra işletim sistemi dilini tekrar Türkçe'ye çevirebilirsiniz."
     exit
   fi

**Explanation**:
  - Checks if the system language is Turkish (`tr_TR`) using `locale`.
  - If detected, prints a warning in Turkish about a WRF bug related to Turkish characters (i vs. ı), suggests changing the language, and exits.


Installing Necessary Packages
-----------------------------
The script installs necessary packages based on the selected WRF type.

.. code-block:: bash

   if [ "$type" = "Chem" ]; then
     extra_packages="flex bison"
   fi
   echo "Please enter your sudo password, so necessary packages can be installed."
   sudo apt-get update
   mpich_repoversion=$(apt-cache policy mpich | grep Candidate | cut -d ':' -f 2 | cut -d '-' -f 1 | cut -c2)
   if [ "$mpich_repoversion" -ge 4 ]; then
       mpirun_packages="libopenmpi-dev libhdf5-openmpi-dev"
   else
       mpirun_packages="mpich libhdf5-mpich-dev"
   fi
   sudo apt-get install -y build-essential csh gfortran m4 curl perl ${mpirun_packages} libpng-dev netcdf-bin libnetcdff-dev ${extra_packages}

**Explanation**:
  - For WRF-Chem (`type="Chem"`), adds `flex` and `bison` to `extra_packages`.
  - Prompts for the sudo password to install packages.
  - `sudo apt-get update`: Updates the package list.
  - Checks the MPICH version in the repository:
    - If version ≥ 4, uses OpenMPI packages (`libopenmpi-dev`, `libhdf5-openmpi-dev`).
    - Otherwise, uses MPICH packages (`mpich`, `libhdf5-mpich-dev`).
  - Installs essential packages (e.g., `gfortran`, `netcdf-bin`) plus any `extra_packages`.



.. code-block:: bash

   package4checks="build-essential csh gfortran m4 curl perl ${mpirun_packages} libpng-dev netcdf-bin libnetcdff-dev ${extra_packages}"
   for packagecheck in ${package4checks}; do
     packagechecked=$(dpkg-query --show --showformat='${db:Status-Status}\n' $packagecheck | grep not-installed)
     if [ "$packagechecked" = "not-installed" ]; then
           echo $packagecheck "$packagechecked"
        packagesnotinstalled=yes
     fi
   done
   if [ "$packagesnotinstalled" = "yes" ]; then
           echo "Some packages were not installed, please re-run the script and enter your root password, when it is requested."
   exit
   fi

**Explanation**:
  - Loops through all required packages to verify installation.
  - `dpkg-query` checks each package’s status; if “not-installed,” sets a flag.
  - If any package is missing, prints an error and exits, prompting the user to rerun with correct permissions.


Set Environment Variables
-------------------------

The script sets environment variables required for the installation.

.. code-block:: bash

   cd ~
   mkdir Build_WRF
   cd Build_WRF
   mkdir LIBRARIES
   cd LIBRARIES

**Explanation**

  - Navigates to the user's home directory (`~`).
  - Creates `Build_WRF` for the installation and a `LIBRARIES` subdirectory for dependency builds.


Configuring Environment Variables
---------------------------------

.. code-block:: bash

   echo "" >> ~/.bashrc
   bashrc_exports=("#WRF Variables" "export DIR=$(pwd)" "export CC=gcc" "export CXX=g++" "export FC=gfortran" "export FCFLAGS=-m64" "export F77=gfortran" "export FFLAGS=-m64"
		   "export NETCDF=/usr" "export HDF5=/usr/lib/x86_64-linux-gnu/hdf5/serial" "export LDFLAGS="\""-L/usr/lib/x86_64-linux-gnu/hdf5/serial/ -L/usr/lib"\""" 
		   "export CPPFLAGS="\""-I/usr/include/hdf5/serial/ -I/usr/include"\""" "export LD_LIBRARY_PATH=/usr/lib")
   for bashrc_export in "${bashrc_exports[@]}" ; do
   [[ -z $(grep "${bashrc_export}" ~/.bashrc) ]] && echo "${bashrc_export}" >> ~/.bashrc
   done
   DIR=$(pwd)
   export CC=gcc
   export CXX=g++
   export FC=gfortran
   export FCFLAGS=-m64
   export F77=gfortran
   export FFLAGS=-m64
   export NETCDF=/usr
   export HDF5=/usr/lib/x86_64-linux-gnu/hdf5/serial
   export LDFLAGS="-L/usr/lib/x86_64-linux-gnu/hdf5/serial/ -L/usr/lib"
   export CPPFLAGS="-I/usr/include/hdf5/serial/ -I/usr/include"
   export LD_LIBRARY_PATH=/usr/lib

- **Explanation**:
  - Adds a blank line to `~/.bashrc` for readability.
  - Defines an array of environment variables (e.g., compilers, library paths) for WRF.
  - Appends these to `~/.bashrc` if not already present.
  - Sets the same variables in the current session for immediate use.


  .. code-block:: bash

   if [ "$type" = "Chem" ]; then
   [[ -z $(grep "export FLEX_LIB_DIR=/usr/lib/x86_64-linux-gnu" ~/.bashrc) ]] && echo "export FLEX_LIB_DIR=/usr/lib/x86_64-linux-gnu" >> ~/.bashrc
   [[ -z $(grep "export YACC='yacc -d'" ~/.bashrc) ]] && echo "export YACC='yacc -d'" >> ~/.bashrc
   export FLEX_LIB_DIR=/usr/lib/x86_64-linux-gnu
   export YACC='yacc -d'
   fi

**Explanation**:
  - For WRF-Chem, adds `FLEX_LIB_DIR` and `YACC` (for `bison`) to `~/.bashrc` and sets them in the current session.


Installing Jasper Library
-------------------------

.. code-block:: bash

   [ -d "jasper-1.900.1" ] && mv jasper-1.900.1 jasper-1.900.1-old
   [ -f "jasper-1.900.1.tar.gz" ] && mv jasper-1.900.1.tar.gz jasper-1.900.1.tar.gz-old
   wget https://www2.mmm.ucar.edu/wrf/OnLineTutorial/compile_tutorial/tar_files/jasper-1.900.1.tar.gz -O jasper-1.900.1.tar.gz
   tar -zxvf jasper-1.900.1.tar.gz
   cd jasper-1.900.1/
   ./configure --prefix=$DIR/grib2
   make
   make install
   [[ -z $(grep "export JASPERLIB=$DIR/grib2/lib" ~/.bashrc) ]] && echo "export JASPERLIB=$DIR/grib2/lib" >> ~/.bashrc
   [[ -z $(grep "export JASPERINC=$DIR/grib2/include" ~/.bashrc) ]] && echo "export JASPERINC=$DIR/grib2/include" >> ~/.bashrc
   export JASPERLIB=$DIR/grib2/lib
   export JASPERINC=$DIR/grib2/include
   cd ..

**Explanation**
  - Moves existing Jasper files to avoid conflicts.
  - Downloads and extracts Jasper 1.900.1, a library for GRIB2 support.
  - Configures and installs it to `$DIR/grib2`.
  - Adds Jasper paths to `~/.bashrc` and the current session.

Installing WRF
--------------

The script downloads and installs the WRF model.


.. code-block:: bash

   cd ..
   [ -d "WRFV${WRFversion}" ] && mv WRFV${WRFversion} WRFV${WRFversion}-old
   [ -f "WRFV${WRFversion}.tar.gz" ] && mv WRFV${WRFversion}.tar.gz WRFV${WRFversion}.tar.gz-old
   wget https://github.com/wrf-model/WRF/releases/download/v${WRFversion}/v${WRFversion}.tar.gz -O WRFV${WRFversion}.tar.gz
   tar -zxvf WRFV${WRFversion}.tar.gz

**Explanation**:
  - Moves existing WRF files and downloads/extracts WRF 4.6.0 source code.


.. code-block:: bash

   if [ "$type" = "Hydro" ]; then
   export WRF_HYDRO=1
   [ -f "v5.3.0.tar.Columns.gz" ] && mv v5.3.0.tar.gz v5.3.0.tar.gz-old
   wget https://github.com/NCAR/wrf_hydro_nwm_public/archive/refs/tags/v5.3.0.tar.gz -O v5.3.0.tar.gz
   tar -zxvf v5.3.0.tar.gz
   /bin/cp -rf wrf_hydro_nwm_public-5.3.0/trunk/NDHMS/* WRFV${WRFversion}/hydro/
   rm v5.3.0.tar.gz
   rm -r wrf_hydro_nwm_public-5.3.0
   fi

- **Explanation**:
  - For WRF-Hydro, enables `WRF_HYDRO`, downloads WRF-Hydro 5.3.0, and integrates it into the WRF hydro directory.


.. code-block:: bash

   cd WRFV${WRFversion}
   if [ "$type" = "Chem" ]; then
   export WRF_CHEM=1
   export WRF_KPP=1
   fi

**Explanation**:
  - For WRF-Chem, enables chemistry (`WRF_CHEM`) and the Kinetic PreProcessor (`WRF_KPP`).


.. code-block:: bash

   sed -i 's#$NETCDF/lib#$NETCDF/lib/x86_64-linux-gnu#g' configure
   ( echo 34 ; echo 1 ) | ./configure
   sed -i 's#-L/usr/lib -lnetcdff -lnetcdf#-L/usr/lib/x86_64-linux-gnu -lnetcdff -lnetcdf#g' configure.wrf

**Explanation**:
  - Adjusts NETCDF paths in `configure` and `configure.wrf`.
  - Configures WRF with options 34 (serial) and 1 (basic nesting).


.. code-block:: bash

   gfortversion=$(gfortran -dumpversion | cut -d '.' -f 1)
   if [ "$gfortversion" -lt 8 ] && [ "$gfortversion" -ge 6 ]; then
   sed -i '/-DBUILD_RRTMG_FAST=1/d' configure.wrf
   fi

- **Explanation**
  - For GFortran versions 6 or 7, removes a problematic flag to ensure compatibility.

.. code-block:: bash

   logsave compile.log ./compile em_real
   if [ -n "$(grep "Problems building executables, look for errors in the build log" compile.log)" ]; then
           echo "Sorry, There were some errors while installing WRF."
           echo "Please create new issue for the problem, https://github.com/bakamotokatas/WRF-Install-Script/issues"
           exit
   fi
   cd ..
   [ -d "WRF-${WRFversion}-${type}" ] && mv WRF-${WRFversion}-${type} WRF-${WRFversion}-${type}-old
   mv WRFV${WRFversion} WRF-${WRFversion}-${type}

**Explanation**:
  - Compiles WRF for real cases, logging to `compile.log`.
  - Checks for errors; if found, exits with a GitHub issue link.
  - Renames the WRF directory with version and type.


Installing WPS (WRF Preprocessing System)
-----------------------------------------

The script downloads and installs the WPS (WRF Preprocessing System).

.. code-block:: bash

   WPSversion="4.6.0"
   [ -d "WPS-${WPSversion}" ] && mv WPS-${WPSversion} WPS-${WPSversion}-old
   [ -f "WPSV${WPSversion}.TAR.gz" ] && mv WPSV${WPSversion}.TAR.gz WPSV${WPSversion}.TAR.gz-old
   wget https://github.com/wrf-model/WPS/archive/v${WPSversion}.tar.gz -O WPSV${WPSversion}.TAR.gz
   tar -zxvf WPSV${WPSversion}.TAR.gz
   cd WPS-${WPSversion}
   ./clean
   sed -i '163s/.*/    NETCDFF="-lnetcdff"/' configure
   sed -i "s/standard_wrf_dirs=.*/standard_wrf_dirs=\"WRF-${WRFversion}-${type} WRF WRF-4.0.3 WRF-4.0.2 WRF-4.0.1 WRF-4.0 WRFV3\"/" configure
   echo 3 | ./configure
   logsave compile.log ./compile
   sed -i "s# geog_data_path.*# geog_data_path = '../WPS_GEOG/'#" namelist.wps
   cd ..

- **Explanation**:
  - Downloads and extracts WPS 4.6.0.
  - Cleans previous builds, adjusts NETCDF and WRF directory settings, configures with option 3 (serial), compiles, and sets the geography path.


Setting Up Geography Data
-------------------------

The script downloads and extracts geographical data files required for WPS.

.. code-block:: bash

   if [ -d "WPS_GEOG" ]; then
     echo "WRF and WPS are installed successfully"
     echo "Directory WPS_GEOG is already exists."
     echo "Do you want WPS_GEOG files to be redownloaded and reextracted?"
     echo "please type yes or no"
     read GEOG_validation
     if [ ${GEOG_validation} = "yes" ]; then
       wget https://www2.mmm.ucar.edu/wrf/src/wps_files/geog_high_res_mandatory.tar.gz -O geog_high_res_mandatory.tar.gz
       tar -zxvf geog_high_res_mandatory.tar.gz
     else
       echo "You can download it later from http://www2.mmm.ucar.edu/wrf/src/wps_files/geog_high_res_mandatory.tar.gz and extract it"
      fi
   else
   wget https://www2.mmm.ucar.edu/wrf/src/wps_files/geog_high_res_mandatory.tar.gz -O geog_high_res_mandatory.tar.gz
   tar -zxvf geog_high_res_mandatory.tar.gz
   fi

**Explanation**:
  - If `WPS_GEOG` exists, asks to redownload geography data; otherwise, downloads and extracts it automatically.


.. code-block:: bash

   if [ "$type" = "Chem" ]; then
    cd WPS_GEOG
    Chem_Geog="modis_landuse_21class_30s soiltype_top_2m soiltype_bot_2m albedo_ncep maxsnowalb erod clayfrac_5m sandfrac_5m"
    for i in ${Chem_Geog}; do
     if [ ! -d $i ]; then
      echo $i
      wget https://www2.mmm.ucar.edu/wrf/src/wps_files/${i}.tar.bz2 -O ${i}.tar.bz2
      tar -xvf ${i}.tar.bz2
      rm ${i}.tar.bz2
     fi
    done
    cd ..
   fi

- **Explanation**:
  - For WRF-Chem, downloads and extracts additional chemistry-related geography datasets.


PREP_CHEM_SRC Installation
--------------------------

The script optionally installs the PREP_CHEM_SRC program for WRF-Chem.

.. code-block:: bash

   if [ "$type" = "Chem" ]; then
    echo "Do you want the PREP-CHEM-SRC program to be installed? PREP-CHEM-SRC is a widely used emission preparation program for WRF-Chem."
    echo "please type yes or no"
    read prep_chem_validation
     if [ ${prep_chem_validation} = "yes" ]; then
     echo "firstly starting to compile convert_emiss.exe. convert_emiss.exe is needed for convert emissions which are created from PREP-CHEM-SRC."
     cd WRF-${WRFversion}-${type}
     logsave convert_emi.log ./compile emi_conv
     cd ..
     echo "Compilation of convert_emiss.exe is finished, now PREP-CHEM-SRC download and compilation has started."
     [ -d "PREP-CHEM-SRC-1.5" ] && mv PREP-CHEM-SRC-1.5 PREP-CHEM-SRC-1.5-old
     [ -f "prep_chem_sources_v1.5_24aug2015.tar.gz" ] && mv prep_chem_sources_v1.5_24aug2015.tar.gz prep_chem_sources_v1.5_24aug2015.tar.gz-old
     wget ftp://aftp.fsl.noaa.gov/divisions/taq/global_emissions/prep_chem_sources_v1.5_24aug2015.tar.gz -O prep_chem_sources_v1.5_24aug2015.tar.gz
     tar -zxvf prep_chem_sources_v1.5_24aug2015.tar.gz
     cd PREP-CHEM-SRC-1.5/bin/build
     sed -i "s#NETCDF=.*#NETCDF=/usr#" include.mk.gfortran.wrf
     sed -i 's#-L$(NETCDF)/lib#-L/usr/lib/x86_64-linux-gnu#' include.mk.gfortran.wrf
     sed -i "s#HDF5=.*#HDF5=/usr/lib/x86_64-linux-gnu/hdf5/serial#" include.mk.gfortran.wrf
     sed -i "s#HDF5_INC=.*#HDF5_INC=-I/usr/include/hdf5/serial#" include.mk.gfortran.wrf
     sed -i 's#-L$(HDF5)/lib#-L/usr/lib/x86_64-linux-gnu/hdf5/serial#' include.mk.gfortran.wrf
     gfortversion=$(gfortran -dumpversion | cut -d '.' -f 1)
     if [ "$gfortversion" -ge 10 ]; then
     sed -i 's#F_OPTS=.*#F_OPTS=  -Xpreprocessor -D$(CHEM) -O2 -fconvert=big-endian -frecord-marker=4 -fallow-argument-mismatch#' include.mk.gfortran.wrf
     fi
     sed -i "s#-L/scratchin/grupos/catt-brams/shared/libs/gfortran/zlib-1.2.8/lib#-L/usr/lib#" include.mk.gfortran.wrf
     sed -i "842s#.*#    'ENERGY     ',\&#" ../../src/edgar_emissions.f90
     sed -i "843s#.*#    'INDUSTRY   ',\&#" ../../src/edgar_emissions.f90
     sed -i "845s#.*#    'TRANSPORT  '/)#" ../../src/edgar_emissions.f90
     make OPT=gfortran.wrf CHEM=RADM_WRF_FIM
     cd ..
     mkdir datain
     cd datain
     wget ftp://aftp.fsl.noaa.gov/divisions/taq/global_emissions/global_emissions_v3_24aug2015.tar.gz -O global_emissions_v3_24aug2015.tar.gz
     tar -zxvf global_emissions_v3_24aug2015.tar.gz
     mv Global_emissions_v3/* .
     rm -r Global_emissions_v3
     mv Emission_data/ EMISSION_DATA
     mv surface_data/ SURFACE_DATA
     cd ../../..
     echo "PREP-CHEM-SRC compilation has finished."
     fi
   fi

**Explanation**:
  - For WRF-Chem, offers to install PREP-CHEM-SRC (emission preparation tool).
  - If “yes,” compiles `convert_emiss.exe`, downloads PREP-CHEM-SRC 1.5, adjusts configuration files, compiles it, and sets up emission data.


Completion
----------

The script completes the installation and exits.

.. code-block:: bash

   echo "Installation has completed"
   exec bash
   exit

**Explanation**:
  - Confirms completion, starts a new Bash session to apply environment changes, and exits.

---