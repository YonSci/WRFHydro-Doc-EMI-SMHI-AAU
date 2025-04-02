
WRF-Hydro Standalone Installation
=================================

.. contents:: On This Page
   :depth: 4

Introduction
------------


.. raw:: html

   <div style="text-align: justify;">
   <p> This document provides a detailed explanation of a bash script designed to download, compile, and install the WRF-Hydro model (version 5.2.0) in standalone mode, along with its necessary prerequisite libraries (like NetCDF, HDF5, MPICH, etc.). It also covers setting up and running a sample test case (Croton_NY). </p>
   </div>


Line-by-Line Explanation
------------------------

This section breaks down the script command by command.

Script Header
~~~~~~~~~~~~~


.. code-block:: bash

   #!/bin/bash

*   ``#!/bin/bash``
        `Shebang line`. Specifies that the script should be executed with bash (Bourne-Again SHell).

Setting Base Directories
~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: bash

   # Set base directory
   BASE_DIR=$HOME/WRFHYDRO_UNCOUPLED
   INSTALL_DIR=$BASE_DIR/libs   # library installation directory
   DOWNLOAD_DIR=$BASE_DIR/Downloads  # library download directory


**Explanation**

    *   ``BASE_DIR=$HOME/WRFHYDRO_UNCOUPLED``
            Defines a variable ``BASE_DIR`` pointing to a directory named ``WRFHYDRO_UNCOUPLED`` within the user's home directory (``$HOME``). This will be the root for all downloads and installations.
    *   ``INSTALL_DIR=$BASE_DIR/libs``
            Defines ``INSTALL_DIR`` where the compiled libraries will be installed.
    *   ``DOWNLOAD_DIR=$BASE_DIR/Downloads``
            Defines ``DOWNLOAD_DIR`` where the source code archives will be downloaded.

Creating Directory Structure
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
.. code-block:: bash

   # Create Directory Structure
   echo "Creating directory structure..."
   mkdir -p $BASE_DIR/libs/NETCDF
   mkdir -p $BASE_DIR/libs/grib2
   mkdir -p $BASE_DIR/libs/MPICH
   mkdir -p $DOWNLOAD_DIR
   cd $BASE_DIR
   # tree -d -L 2 # Optional visualization

**Explanation**

    *   ``echo "Creating..."``
            Prints a status message to the terminal.
    *   ``mkdir -p ...``
            Creates the specified directories. The ``-p`` flag ensures that parent directories are created if they don't exist. Specific subdirectories for NetCDF, grib2-related libraries (like HDF5, zlib), and MPICH are created within the installation directory. The download directory is also created.
    *   ``cd $BASE_DIR``
            Changes the current working directory to the base directory for further operations.
    *   ``# tree -d -L 2``
            A commented-out command. If uncommented and the ``tree`` command is installed, it would display the created directory structure up to 2 levels deep.

Downloading Prerequisite Libraries
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: bash

   # Download Libraries
   echo "Downloading libraries..."

   cd $DOWNLOAD_DIR

   # Download zlib Library
   wget -c -4 https://github.com/madler/zlib/archive/refs/tags/v1.2.12.tar.gz

   # Download HDF Library
   wget -c -4 https://github.com/HDFGroup/hdf5/archive/refs/tags/hdf5-1_12_2.tar.gz

   # Download netCDF-C Library
   wget -c -4 https://github.com/Unidata/netcdf-c/archive/refs/tags/v4.9.0.tar.gz

   # Download netCDF-Fortran Library
   wget -c -4 https://github.com/Unidata/netcdf-fortran/archive/refs/tags/v4.6.0.tar.gz

   # Download MPICH Library
   wget -c -4 https://github.com/pmodels/mpich/releases/download/v4.0.2/mpich-4.0.2.tar.gz

   # Download libpng Library
   wget -c -4 https://download.sourceforge.net/libpng/libpng-1.6.37.tar.gz

   # Download jasper Library
   wget -c -4 https://www.ece.uvic.ca/~frodo/jasper/software/jasper-1.900.1.zip

   # Download szip Library (Placeholder - see note in full script)
   # wget -c -4 [URL_TO_SZIP_TARBALL]

**Explanation**

    *   ``echo "Downloading..."``
            Prints a status message.
    *   ``cd $DOWNLOAD_DIR``
            Changes the current directory to where downloads should be stored.
    *   ``wget -c -4 [URL]``:  Downloads files from the specified URLs.
        *   ``wget``: A command-line utility to retrieve files from the web.  
        *   ``-c``: Continue getting a partially downloaded file.  
        *   ``-4``: Force resolving hostnames to IPv4 addresses only.  


    *   The script downloads source code archives for:

            *   zlib (compression library),
            *   HDF5 (data format), 
            *   NetCDF-C (data format C library),
            *   NetCDF-Fortran (Fortran interface), 
            *   MPICH (MPI implementation), 
            *   libpng (PNG image format), 
            *   Jasper (JPEG-2000 format), and 
            *   Szip (another compression library often used with HDF5).


Setting Compilers
~~~~~~~~~~~~~~~~~~


.. code-block:: bash

   # Setting Compilers
   echo "Setting compilers..."
   export CC=gcc
   export CXX=g++
   export FC=gfortran
   export F77=gfortran


**Explanation**

*   ``echo "Setting compilers..."``: Prints a status message.
*   ``export CC=gcc``: Sets the environment variable ``CC`` (C compiler) to ``gcc``.
*   ``export CXX=g++``: Sets the environment variable ``CXX`` (C++ compiler) to ``g++``.
*   ``export FC=gfortran``: Sets the environment variable ``FC`` (Fortran compiler, typically Fortran 90+) to ``gfortran``.
*   ``export F77=gfortran``: Sets the environment variable ``F77`` (Fortran 77 compiler) to ``gfortran`` . These environment variables are commonly used by build systems (like ``configure`` scripts) to determine which compilers to use.

Checking Compiler Versions
~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: bash

   # Print Compiler Versions
   echo "Checking compiler versions..."
   gcc_version="$(gcc -dumpversion)"
   gfortran_version="$(gfortran -dumpversion)"
   gplusplus_version="$(g++ -dumpversion)"
   echo "GCC: $gcc_version, GFortran: $gfortran_version, G++: $gplusplus_version"

**Explanation**

*   ``echo "Checking..."``: Prints a status message.
*   ``gcc_version="$(gcc -dumpversion)"``: Executes ``gcc -dumpversion`` (which prints the compiler version) and captures its output into the bash variable ``gcc_version``. Similar commands capture versions for ``gfortran`` and ``g++``.
*   ``echo "GCC: ..."``: Prints the detected compiler versions to the terminal.

Setting Compiler Flags Based on Version
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: bash

   # Set Compiler Flags based on version
   echo "Setting compiler flags..."
   export version_10="10"
   if [ "$gcc_version" != "" ] && [ "$(echo $gcc_version | cut -d. -f1)" -ge "$version_10" ] || \
      [ "$gfortran_version" != "" ] && [ "$(echo $gfortran_version | cut -d. -f1)" -ge "$version_10" ] || \
      [ "$gplusplus_version" != "" ] && [ "$(echo $gplusplus_version | cut -d. -f1)" -ge "$version_10" ]
   then
     export fallow_argument="-fallow-argument-mismatch"
     export boz_argument="-fallow-invalid-boz" # Note usage
   else
     export fallow_argument=""
     export boz_argument=""
   fi

   export FFLAGS="$fallow_argument $boz_argument"
   export FCFLAGS="$fallow_argument $boz_argument"

**Explanation**

*   ``echo "Setting flags..."``: Prints a status message. 

*  ``export version_10="10"``: Sets a variable for version 10.
*  The ``if`` condition checks if any compiler version is 10 or higher:
    *  If true, sets ``fallow_argument`` to ``-fallow-argument-mismatch`` and ``boz_argument`` to ``-fallow-invalid-boz`` (for compatibility).
    *  If false, sets both to empty strings.

*  ``export FFLAGS="$fallow_argument"``: Sets Fortran flags.
*  ``export FCFLAGS="$fallow_argument"``: Sets Fortran compiler flags.


Installing zlib
~~~~~~~~~~~~~~~

.. code-block:: bash

   # Install zlib
   echo "Installing zlib..."
   cd $DOWNLOAD_DIR
   tar -xvzf v1.2.12.tar.gz
   cd zlib-1.2.12/
   DIR=$INSTALL_DIR
   CC= CXX= ./configure --prefix=$DIR/grib2
   make
   make install

**Explanation**

  - ``echo "Installing zlib..."``: Prints a status message.
  - ``cd $DOWNLOAD_DIR``: Navigates to the download directory.
  - ``tar -xvzf v1.2.12.tar.gz``: Extracts the zlib archive.
  - ``cd zlib-1.2.12/``: Enters the extracted directory.
  - ``DIR=$INSTALL_DIR``: Sets a variable for the install directory.
  - ``CC= CXX= ./configure --prefix=$DIR/grib2``: Configures zlib with an empty compiler setting (likely a mistake; compilers are set earlier) and installs to ``$DIR/grib2``.
  - ``make``: Compiles zlib.
  - ``make install``: Installs zlib.


Installing libpng
~~~~~~~~~~~~~~~~~

.. code-block:: bash

   # Install libpng
   echo "Installing libpng..."
   cd $DOWNLOAD_DIR
   export LDFLAGS=-L$DIR/grib2/lib
   export CPPFLAGS=-I$DIR/grib2/includeS
   tar -xvzf libpng-1.6.37.tar.gz
   cd libpng-1.6.37/
   ./configure --prefix=$DIR/grib2
   make
   make install

**Explanation**

    - ``echo "Installing libpng..."``: Prints a status message.
    - ``cd $DOWNLOAD_DIR``: Navigates to the download directory.
    - ``export LDFLAGS=-L$DIR/grib2/lib``: Sets linker flags to include zlib’s library path.
    - ``export CPPFLAGS=-I$DIR/grib2/includeS``: Sets preprocessor flags; ``includeS`` is a typo (should be ``include``).
    - ``tar -xvzf libpng-1.6.37.tar.gz``: Extracts libpng.
    - ``cd libpng-1.6.37/``: Enters the extracted directory.
    - ``./configure --prefix=$DIR/grib2``: Configures libpng to install in ``$DIR/grib2``.
    - ``make``: Compiles libpng.
    - ``make install``: Installs libpng.


Installing Jasper
~~~~~~~~~~~~~~~~~

.. code-block:: bash

   # Install Jasper
   echo "Installing Jasper..."
   cd $DOWNLOAD_DIR
   unzip jasper-1.900.1.zip
   cd jasper-1.900.1/
   autoreconf -i
   ./configure --prefix=$DIR/grib2
   make
   make install
   export JASPERLIB=$DIR/grib2/lib
   export JASPERINC=$DIR/grib2/include

**Explanation**
    - ``echo "Installing Jasper..."``: Prints a status message.
    - ``cd $DOWNLOAD_DIR``: Navigates to the download directory.
    - ``unzip jasper-1.900.1.zip``: Extracts the Jasper zip file.
    - ``cd jasper-1.900.1/``: Enters the extracted directory.
    - ``autoreconf -i``: Generates the configure script.
    - ``./configure --prefix=$DIR/grib2``: Configures Jasper to install in ``$DIR/grib2``.
    - ``make``: Compiles Jasper.
    - ``make install``: Installs Jasper.
    - ``export JASPERLIB=$DIR/grib2/lib``: Sets the Jasper library path.
    - ``export JASPERINC=$DIR/grib2/include``: Sets the Jasper include path.

Installing MPICH
~~~~~~~~~~~~~~~~~

.. code-block:: bash

   # Install MPICH
   echo "Installing MPICH..."
   cd $DOWNLOAD_DIR
   tar -xvzf mpich-4.0.2.tar.gz
   cd mpich-4.0.2/
   F90= ./configure --prefix=$DIR/MPICH --with-device=ch3 FFLAGS=$fallow_argument FCFLAGS=$fallow_argument
   make
   make install
   export PATH=$DIR/MPICH/bin:$PATH

**Explanation**

    - ``echo "Installing MPICH..."``: Prints a status message.
    - ``cd $DOWNLOAD_DIR``: Navigates to the download directory.
    - ``tar -xvzf mpich-4.0.2.tar.gz``: Extracts MPICH.
    - ``cd mpich-4.0.2/``: Enters the extracted directory.
    - ``F90= ./configure --prefix=$DIR/MPICH --with-device=ch3 FFLAGS=$fallow_argument FCFLAGS=$fallow_argument``: Configures MPICH with no F90 compiler specified, installs to ``$DIR/MPICH``, uses the ``ch3`` device, and applies compiler flags.
    - ``make``: Compiles MPICH.
    - ``make install``: Installs MPICH.
    - ``export PATH=$DIR/MPICH/bin:$PATH``: Adds MPICH’s bin directory to the ``PATH``.


Installing Szip
~~~~~~~~~~~~~~~

.. code-block:: bash

   # Install szip
   echo "Installing Szip"
   cd $DOWNLOAD_DIR
   tar -xvzf szip-2.1.tar.gz
   cd szip-2.1
   ./configure --prefix=$HOME/WRF/libs/szip
   make   # compile the code
   make install # installl szip

**Explanation**

    - ``echo "Installing Szip"``: Prints a status message.
    - ``cd $DOWNLOAD_DIR``: Navigates to the download directory.
    - ``tar -xvzf szip-2.1.tar.gz``: Extracts Szip.
    - ``cd szip-2.1``: Enters the extracted directory.
    - ``./configure --prefix=$HOME/WRF/libs/szip``: Configures Szip to install in ``$HOME/WRF/libs/szip`` (note the different path).
    - ``make``: Compiles Szip.
    - ``make install``: Installs Szip (comment has a typo: "installl").


Installing HDF5
~~~~~~~~~~~~~~~

.. code-block:: bash

   # Install HDF5
   echo "Installing HDF5..."
   cd $DOWNLOAD_DIR
   tar -xvzf hdf5-1_12_2.tar.gz
   cd hdf5-hdf5-1_12_2
   ./configure --prefix=$DIR/grib2 --with-zlib=$DIR/grib2 --enable-hl --enable-fortran
   make
   make install
   export HDF5=$DIR/grib2
   export LD_LIBRARY_PATH=$DIR/grib2/lib:$LD_LIBRARY_PATH

**Explanation**

    - ``echo "Installing HDF5..."``: Prints a status message.
    - ``cd $DOWNLOAD_DIR``: Navigates to the download directory.
    - ``tar -xvzf hdf5-1_12_2.tar.gz``: Extracts HDF5.
    - ``cd hdf5-hdf5-1_12_2``: Enters the extracted directory.
    - ``./configure --prefix=$DIR/grib2 --with-zlib=$DIR/grib2 --enable-hl --enable-fortran``: Configures HDF5 with zlib support, high-level API, and Fortran support, installing to ``$DIR/grib2``.
    - ``make``: Compiles HDF5.
    - ``make install``: Installs HDF5.
    - ``export HDF5=$DIR/grib2``: Sets the HDF5 environment variable.
    - ``export LD_LIBRARY_PATH=$DIR/grib2/lib:$LD_LIBRARY_PATH``: Adds HDF5’s library path to ``LD_LIBRARY_PATH``.


Installing netCDF-C
~~~~~~~~~~~~~~~~~~~~

.. code-block:: bash

   # Install netCDF-C
   echo "Installing netCDF-C..."
   cd $DOWNLOAD_DIR
   export CPPFLAGS=-I$DIR/grib2/include
   export LDFLAGS=-L$DIR/grib2/lib
   tar -xzvf v4.9.0.tar.gz
   cd netcdf-c-4.9.0/
   ./configure --prefix=$DIR/NETCDF --disable-dap
   make
   make install
   export PATH=$DIR/NETCDF/bin:$PATH
   export NETCDF=$DIR/NETCDF

**Explanation**

    - ``echo "Installing netCDF-C..."``: Prints a status message.
    - ``cd $DOWNLOAD_DIR``: Navigates to the download directory.
    - ``export CPPFLAGS=-I$DIR/grib2/include``: Sets include path for grib2.
    - ``export LDFLAGS=-L$DIR/grib2/lib``: Sets library path for grib2.
    - ``tar -xzvf v4.9.0.tar.gz``: Extracts netCDF-C.
    - ``cd netcdf-c-4.9.0/``: Enters the extracted directory.
    - ``./configure --prefix=$DIR/NETCDF --disable-dap``: Configures netCDF-C to install in ``$DIR/NETCDF`` and disables DAP.
    - ``make``: Compiles netCDF-C.
    - ``make install``: Installs netCDF-C.
    - ``export PATH=$DIR/NETCDF/bin:$PATH``: Adds netCDF-C’s bin directory to ``PATH``.
    - ``export NETCDF=$DIR/NETCDF``: Sets the NETCDF environment variable.


Installing netCDF-Fortran
~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: bash

   # Install netCDF-Fortran
   echo "Installing netCDF-Fortran..."
   cd $DOWNLOAD_DIR
   tar -xvzf v4.6.0.tar.gz
   cd netcdf-fortran-4.6.0/
   export LD_LIBRARY_PATH=$DIR/NETCDF/lib:$LD_LIBRARY_PATH
   export CPPFLAGS=-I$DIR/NETCDF/include
   export LDFLAGS=-L$DIR/NETCDF/lib
   ./configure --prefix=$DIR/NETCDF --disable-shared
   make
   make install

**Explanation**

    - ``echo "Installing netCDF-Fortran..."``: Prints a status message.
    - ``cd $DOWNLOAD_DIR``: Navigates to the download directory.
    - ``tar -xvzf v4.6.0.tar.gz``: Extracts netCDF-Fortran.
    - ``cd netcdf-fortran-4.6.0/``: Enters the extracted directory.
    - ``export LD_LIBRARY_PATH=$DIR/NETCDF/lib:$LD_LIBRARY_PATH``: Adds netCDF library path.
    - ``export CPPFLAGS=-I$DIR/NETCDF/include``: Sets netCDF include path.
    - ``export LDFLAGS=-L$DIR/NETCDF/lib``: Sets netCDF library path.
    - ``./configure --prefix=$DIR/NETCDF --disable-shared``: Configures netCDF-Fortran to install in ``$DIR/NETCDF`` without shared libraries.
    - ``make``: Compiles netCDF-Fortran.
    - ``make install``: Installs netCDF-Fortran.



Exporting Paths to .bashrc
~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: bash

   # Appending PATH to .bashrc
   echo "Exporting necessary paths to .bashrc..."
   echo "export PATH=$INSTALL_DIR/MPICH/bin:\$PATH" >> $HOME/.bashrc
   echo "export PATH=$INSTALL_DIR/NETCDF/bin:\$PATH" >> $HOME/.bashrc
   echo "export LD_LIBRARY_PATH=$INSTALL_DIR/NETCDF/lib:\$LD_LIBRARY_PATH" >> $HOME/.bashrc
   echo "export HDF5=$INSTALL_DIR/grib2" >> $HOME/.bashrc
   echo "export LD_LIBRARY_PATH=$INSTALL_DIR/grib2/lib:\$LD_LIBRARY_PATH" >> $HOME/.bashrc
   echo "export NETCDF=$INSTALL_DIR/NETCDF" >> $HOME/.bashrc
   source $HOME/.bashrc

**Explanation**

    - ``echo "Exporting necessary paths to .bashrc..."``: Prints a status message.
    - Each ``echo "export ..." >> $HOME/.bashrc`` appends an export command to ``.bashrc``:
        - Adds MPICH and NETCDF bin directories to ``PATH``.
        - Adds NETCDF and grib2 library paths to ``LD_LIBRARY_PATH``.
        - Sets ``HDF5`` and ``NETCDF`` environment variables.
    - ``source $HOME/.bashrc``: Applies these changes to the current session.


Download and Configure WRF-Hydro
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: bash

   # Download and Configure WRF-Hydro
   echo "Setting up WRF-Hydro..."
   cd $DOWNLOAD_DIR
   wget -c https://github.com/NCAR/wrf_hydro_nwm_public/archive/refs/tags/v5.2.0.tar.gz -O WRFHYDRO.5.2.tar.gz
   mkdir -p $HOME/WRFhydromodel/domain/NWM
   tar -xvzf WRFHYDRO.5.2.tar.gz -C $HOME/WRFhydromodel

   # Configure WRF-Hydro Environment Settings
   cd $HOME/WRFhydromodel/wrf_hydro_nwm_public-5.2.0/trunk/NDHMS/template
   cp setEnvar.sh setEnvar.sh.orig
   sed 's/SPATIAL_SOIL=0/SPATIAL_SOIL=1/' setEnvar.sh.orig > setEnvar.sh
   echo "" >> setEnvar.sh
   echo "# Large netcdf file support: 0=Off, 1=On." >> setEnvar.sh
   echo "export WRFIO_NCD_LARGE_FILE_SUPPORT=1" >> setEnvar.sh
   ln -sf $PWD/setEnvar.sh $HOME/WRFhydromodel/wrf_hydro_nwm_public-5.2.0/trunk/NDHMS/setEnvar.sh

**Explanation**

*   ``echo "Setting up..."``: Prints a status message.
*   ``cd $DOWNLOAD_DIR``: Navigates to the download directory.
*   ``wget ... -O WRFHYDRO.5.2.tar.gz``: Downloads the WRF-Hydro v5.2.0 source code, saving it with a specific filename ``WRFHYDRO.5.2.tar.gz``.
*   ``mkdir -p $HOME/WRFhydromodel/domain/NWM``: Creates the directory structure where the model will eventually be run (the domain/test case directory).
*   ``tar -xvzf ... -C $HOME/WRFhydromodel``: Extracts the WRF-Hydro source code into the ``$HOME/WRFhydromodel`` directory.
*   ``cd .../template``: Navigates into the template directory within the WRF-Hydro source.
*   ``cp setEnvar.sh setEnvar.sh.orig``: Creates a backup of the original environment settings file.
*   ``sed 's/...' ... > setEnvar.sh``: Modifies the environment settings file (``setEnvar.sh``). This specific command enables the ``SPATIAL_SOIL`` option by changing its value from 0 to 1.
*   ``echo ... >> setEnvar.sh``: Appends lines to the ``setEnvar.sh`` file to add a comment and enable large NetCDF file support via the ``WRFIO_NCD_LARGE_FILE_SUPPORT`` environment variable.
*   ``ln -sf ...``: Creates a symbolic link from the modified ``setEnvar.sh`` in the template directory to the main compilation directory (``.../trunk/NDHMS``), ensuring the compile script uses the modified settings.

Compile WRF-Hydro
~~~~~~~~~~~~~~~~~

.. code-block:: bash

   # Compile WRF-Hydro
   echo "Compiling WRF-Hydro..."
   cd $HOME/WRFhydromodel/wrf_hydro_nwm_public-5.2.0/trunk/NDHMS
   ./configure # Option 2
   if [ $? -ne 0 ]; then echo "WRF-Hydro configure failed!"; exit 1; fi
   ./compile_offline_NoahMP.sh setEnvar.sh
   if [ $? -ne 0 ]; then echo "WRF-Hydro compile failed!"; exit 1; fi


**Explanation**


*   ``echo "Compiling..."``: Prints a status message.
*   ``cd .../NDHMS``: Navigates to the main WRF-Hydro source/compile directory.
*   ``./configure``: Runs the WRF-Hydro configure script. This script typically prompts the user to select compilation options (like compiler choice, nesting options). The comment ``# Option 2`` suggests the user should manually select option 2 when prompted.
*   ``if [ $? -ne 0 ]; then ... fi``: Basic error checking. If the previous command (``./configure``) exited with a non-zero status (indicating an error), it prints an error message and exits the script.
*   ``./compile_offline_NoahMP.sh setEnvar.sh``: Executes the specific compilation script for the standalone ("offline") NoahMP configuration, passing the (modified) ``setEnvar.sh`` file to provide necessary environment settings (like library paths) to the build system.
*   ``if [ $? -ne 0 ]; then ... fi``: Checks for errors after the compilation step.

Copy Compiled Files to Run Directory
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: bash

   # Copy .TBL and Executables to Run Directory
   echo "Copying .TBL and executables..."
   DEST_RUN_DIR="$HOME/WRFhydromodel/domain/NWM"
   cp Run/*.TBL $DEST_RUN_DIR
   cp Run/wrf_hydro.exe $DEST_RUN_DIR

**Explanation**

*   ``echo "Copying..."``: Prints a status message.
*   ``DEST_RUN_DIR="..."``: Defines the destination run directory variable again for clarity.
*   ``cp Run/*.TBL $DEST_RUN_DIR``: Copies all files ending with ``.TBL`` (table files containing parameterizations or settings) from the ``Run`` subdirectory (within the compile directory) to the designated run directory.
*   ``cp Run/wrf_hydro.exe $DEST_RUN_DIR``: Copies the compiled WRF-Hydro executable (``wrf_hydro.exe``) from the ``Run`` subdirectory to the run directory.

Download and Extract Test Case
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: bash

   # Download Croton_NY Test Case
   echo "Downloading Croton_NY Test Case..."
   cd $DOWNLOAD_DIR
   wget -c https://github.com/NCAR/wrf_hydro_nwm_public/releases/download/v5.2.0/croton_NY_training_example_v5.2.tar.gz

   # Extracting the tar.gz file
   echo "Extracting Croton_NY Test Case..."
   tar -xvzf croton_NY_training_example_v5.2.tar.gz -C $DOWNLOAD_DIR

**Explanation**


*   ``echo "Downloading..."``: Prints a status message.
*   ``cd $DOWNLOAD_DIR``: Changes to the download directory.
*   ``wget ...``: Downloads the Croton_NY test case archive.
*   ``echo "Extracting..."``: Prints a status message.
*   ``tar -xvzf ... -C $DOWNLOAD_DIR``: Extracts the test case archive into the download directory. It will likely create a subdirectory like ``example_case``.

Copy Test Case Files to Run Directory
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: bash

   # Defining the source and destination paths for training examples
   SOURCE_CASE_DIR="$DOWNLOAD_DIR/example_case" # Path after extraction
   DEST_RUN_DIR="$HOME/WRFhydromodel/domain/NWM" # Defined earlier

   # Copying Test Case Files to Run Directory
   echo "Copying Test Case files..."
   cp -r $SOURCE_CASE_DIR/FORCING $DEST_RUN_DIR/
   cp -r $SOURCE_CASE_DIR/NWM/DOMAIN $DEST_RUN_DIR/
   cp -r $SOURCE_CASE_DIR/NWM/RESTART $DEST_RUN_DIR/
   cp -r $SOURCE_CASE_DIR/NWM/nudgingTimeSliceObs $DEST_RUN_DIR/
   # cp -r $SOURCE_CASE_DIR/NWM/referenceSim $DEST_RUN_DIR # Usually not needed for running
   cp $SOURCE_CASE_DIR/NWM/namelist.hrldas $DEST_RUN_DIR/
   cp $SOURCE_CASE_DIR/NWM/hydro.namelist $DEST_RUN_DIR/

**Explanation**

*   ``SOURCE_CASE_DIR="..."``: Defines a variable pointing to the extracted test case directory.
*   ``DEST_RUN_DIR="..."``: Re-affirms the run directory variable.
*   ``echo "Copying..."``: Prints a status message.
*   ``cp -r ... $DEST_RUN_DIR/``: Copies the necessary directories (``FORCING``, ``DOMAIN``, ``RESTART``, ``nudgingTimeSliceObs``) recursively (``-r``) from the extracted test case source into the run directory.
*   ``cp ... $DEST_RUN_DIR/``: Copies the required namelist files (``namelist.hrldas``, ``hydro.namelist``) into the run directory. These files contain settings and configurations for the specific model run.

Verify Copied Files
~~~~~~~~~~~~~~~~~~~

.. code-block:: bash

   # Verifying the copied files and directories
   echo "Verifying contents of run directory:"
   ls -R $DEST_RUN_DIR | head -n 20 # Show partial listing

**Explanation**

*   ``echo "Verifying..."``: Prints a status message.
*   ``ls -R $DEST_RUN_DIR | head -n 20``: Lists the contents of the run directory recursively (``-R``) and pipes (``|``) the output to ``head -n 20``, which displays only the first 20 lines. This gives a quick check that files were copied.

Navigate to Run Directory
~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: bash

   # Navigating to the Run Directory
   echo "Navigating to run directory: $DEST_RUN_DIR"
   cd $DEST_RUN_DIR

**Explanation**

*   ``echo "Navigating..."``: Prints a status message indicating the directory change.
*   ``cd $DEST_RUN_DIR``: Changes the current working directory to the run directory where the executable and test case files are located.

Run WRF-Hydro
~~~~~~~~~~~~~~~

.. code-block:: bash

   # Running WRF-Hydro with MPI
   echo "Running WRF-Hydro using mpirun..."
   mpirun -np 2 ./wrf_hydro.exe
   if [ $? -ne 0 ]; then echo "WRF-Hydro run command failed!"; else echo "WRF-Hydro run command finished."; fi

   # Optional: Running WRF-Hydro while capturing output/errors
   # echo "Running WRF-Hydro with logging..."
   # mpirun -np 2 ./wrf_hydro.exe > wrf_hydro_run.log 2>&1

**Explanation**

*   ``echo "Running..."``: Prints a status message.
*   ``mpirun -np 2 ./wrf_hydro.exe``: Executes the WRF-Hydro model.
    *   ``mpirun``: A command (typically provided by an MPI implementation like MPICH or OpenMPI) used to launch parallel applications.
    *   ``-np 2``: Specifies the number of processes (cores) to use (in this case, 2).
    *   ``./wrf_hydro.exe``: The executable file to run.
*   ``if [ $? -ne 0 ]; then ... fi``: Checks the exit status of the ``mpirun`` command and prints a success or failure message.
*   The commented-out lines show how to redirect standard output (``>``) and standard error (``2>&1``) to a log file (``wrf_hydro_run.log``) for later inspection.

Verify Output
~~~~~~~~~~~~~

.. code-block:: bash

   # Verifying the Output
   echo "Checking for output files (HYDRO_RST*)..."
   ls -lah HYDRO_RST*

   echo "Script finished."

**Explanation**

*   ``echo "Checking..."``: Prints a status message.
*   ``ls -lah HYDRO_RST*``: Lists files in the current directory (the run directory) that start with ``HYDRO_RST``. These are typically the hydrostatic restart output files generated by a successful model run.
    *   ``-l``: Long listing format.
    *   ``-a``: Show all files (including hidden).
    *   ``-h``: Human-readable file sizes.
*   ``echo "Script finished."``: Prints a final message indicating the script has complete






The Full Script
---------------

Here is the full Bash script for compiling and running the WRF-Hydro model in standalone mode. It includes downloading and installing prerequisite libraries and setting up a sample test case in a single run.


.. code-block:: bash
   :caption: Full Installation Script
   :linenos:

   #!/bin/bash

    # WRF-Hydro Model Installation in Standalone Mode

    # Set base directory
    BASE_DIR=$HOME/WRFHYDRO_UNCOUPLED	  
    INSTALL_DIR=$BASE_DIR/libs   # library installation directory
    DOWNLOAD_DIR=$BASE_DIR/Downloads  # library download directory

    # Create Directory Structure
    echo "Creating directory structure..."
    mkdir -p $BASE_DIR/libs/NETCDF
    mkdir -p $BASE_DIR/libs/grib2
    mkdir -p $BASE_DIR/libs/MPICH
    midir -p $BASE_DIR/libs/MPICH
    mkdir -p $DOWNLOAD_DIR
    cd $BASE_DIR
    tree -d -L 2

    # Download Libraries
    echo "Downloading libraries..."

    cd $DOWNLOAD_DIR

    # Download zlib Library
    wget -c -4 https://github.com/madler/zlib/archive/refs/tags/v1.2.12.tar.gz

    # Download HDF Library
    wget -c -4 https://github.com/HDFGroup/hdf5/archive/refs/tags/hdf5-1_12_2.tar.gz

    # Download netCDF-C Library
    wget -c -4 https://github.com/Unidata/netcdf-c/archive/refs/tags/v4.9.0.tar.gz

    # Download netCDF-Fortran Library
    wget -c -4 https://github.com/Unidata/netcdf-fortran/archive/refs/tags/v4.6.0.tar.gz

    # Download MPICH Library
    wget -c -4 https://github.com/pmodels/mpich/releases/download/v4.0.2/mpich-4.0.2.tar.gz

    # Download libpng Library
    wget -c -4 https://download.sourceforge.net/libpng/libpng-1.6.37.tar.gz

    # Download jasper Library
    wget -c -4 https://www.ece.uvic.ca/~frodo/jasper/software/jasper-1.900.1.zip

    # Download szip Libarary
    wget -c -4 https://www.gidhome.com/ftp/pub/gid_adds/binaries/szip-2.1/szip-2.1.tar.gz

    # Setting Compilers
    echo "Setting compilers..."
    export CC=gcc
    export CXX=g++
    export FC=gfortran
    export F77=gfortran

    # Print Compiler Versions
    echo "Checking compiler versions..."
    gcc_version="$(gcc -dumpversion)"
    gfortran_version="$(gfortran -dumpversion)"
    gplusplus_version="$(g++ -dumpversion)"
    echo "GCC: $gcc_version, GFortran: $gfortran_version, G++: $gplusplus_version"

    # Set Compiler Flags based on version
    echo "Setting compiler flags..."
    export version_10="10"
    if [ $gcc_version -ge $version_10 ] || [ $gfortran_version -ge $version_10 ] || [ $gplusplus_version -ge $version_10 ]
    then
    export fallow_argument="-fallow-argument-mismatch"
    export boz_argument="-fallow-invalid-boz"
    else 
    export fallow_argument=""
    export boz_argument=""
    fi

    export FFLAGS="$fallow_argument"
    export FCFLAGS="$fallow_argument"

    # Install zlib
    echo "Installing zlib..."
    cd $DOWNLOAD_DIR
    tar -xvzf v1.2.12.tar.gz
    cd zlib-1.2.12/
    DIR=$INSTALL_DIR
    CC= CXX= ./configure --prefix=$DIR/grib2
    make
    make install

    # Install libpng
    echo "Installing libpng..."
    cd $DOWNLOAD_DIR
    export LDFLAGS=-L$DIR/grib2/lib
    export CPPFLAGS=-I$DIR/grib2/includeS
    tar -xvzf libpng-1.6.37.tar.gz
    cd libpng-1.6.37/
    ./configure --prefix=$DIR/grib2
    make
    make install

    # Install Jasper
    echo "Installing Jasper..."
    cd $DOWNLOAD_DIR
    unzip jasper-1.900.1.zip
    cd jasper-1.900.1/
    autoreconf -i
    ./configure --prefix=$DIR/grib2
    make
    make install
    export JASPERLIB=$DIR/grib2/lib
    export JASPERINC=$DIR/grib2/include

    # Install MPICH
    echo "Installing MPICH..."
    cd $DOWNLOAD_DIR
    tar -xvzf mpich-4.0.2.tar.gz
    cd mpich-4.0.2/
    F90= ./configure --prefix=$DIR/MPICH --with-device=ch3 FFLAGS=$fallow_argument FCFLAGS=$fallow_argument
    make
    make install
    export PATH=$DIR/MPICH/bin:$PATH


    # Install szip

    echo "Installing Szip"
    cd $DOWNLOAD_DIR
    tar -xvzf szip-2.1.tar.gz
    cd szip-2.1
    ./configure --prefix=$HOME/WRF/libs/szip
    make   #compile the code
    make install # installl szip 

    # Install HDF5
    echo "Installing HDF5..."
    cd $DOWNLOAD_DIR
    tar -xvzf hdf5-1_12_2.tar.gz
    cd hdf5-hdf5-1_12_2
    ./configure --prefix=$DIR/grib2 --with-zlib=$DIR/grib2 --enable-hl --enable-fortran
    make
    make install
    export HDF5=$DIR/grib2
    export LD_LIBRARY_PATH=$DIR/grib2/lib:$LD_LIBRARY_PATH

    # Install netCDF-C
    echo "Installing netCDF-C..."
    cd $DOWNLOAD_DIR
    export CPPFLAGS=-I$DIR/grib2/include 
    export LDFLAGS=-L$DIR/grib2/lib
    tar -xzvf v4.9.0.tar.gz
    cd netcdf-c-4.9.0/
    ./configure --prefix=$DIR/NETCDF --disable-dap
    make
    make install
    export PATH=$DIR/NETCDF/bin:$PATH
    export NETCDF=$DIR/NETCDF

    # Install netCDF-Fortran
    echo "Installing netCDF-Fortran..."
    cd $DOWNLOAD_DIR
    tar -xvzf v4.6.0.tar.gz
    cd netcdf-fortran-4.6.0/
    export LD_LIBRARY_PATH=$DIR/NETCDF/lib:$LD_LIBRARY_PATH
    export CPPFLAGS=-I$DIR/NETCDF/include 
    export LDFLAGS=-L$DIR/NETCDF/lib
    ./configure --prefix=$DIR/NETCDF --disable-shared
    make
    make install


    # Appending PATH to .bashrc
    echo "Exporting necessary paths to .bashrc..."
    echo "export PATH=$INSTALL_DIR/MPICH/bin:\$PATH" >> $HOME/.bashrc
    echo "export PATH=$INSTALL_DIR/NETCDF/bin:\$PATH" >> $HOME/.bashrc
    echo "export LD_LIBRARY_PATH=$INSTALL_DIR/NETCDF/lib:\$LD_LIBRARY_PATH" >> $HOME/.bashrc
    echo "export HDF5=$INSTALL_DIR/grib2" >> $HOME/.bashrc
    echo "export LD_LIBRARY_PATH=$INSTALL_DIR/grib2/lib:\$LD_LIBRARY_PATH" >> $HOME/.bashrc
    echo "export NETCDF=$INSTALL_DIR/NETCDF" >> $HOME/.bashrc

    # Apply changes to current session
    source $HOME/.bashrc

    # Download and Configure WRF-Hydro
    echo "Setting up WRF-Hydro..."
    cd $DOWNLOAD_DIR
    wget -c https://github.com/NCAR/wrf_hydro_nwm_public/archive/refs/tags/v5.2.0.tar.gz -O WRFHYDRO.5.2.tar.gz
    mkdir -p $HOME/WRFhydromodel
    tar -xvzf WRFHYDRO.5.2.tar.gz -C $HOME/WRFhydromodel

    cd $HOME/WRFhydromodel/wrf_hydro_nwm_public-5.2.0/trunk/NDHMS/template
    sed -i 's/SPATIAL_SOIL=0/SPATIAL_SOIL=1/g' setEnvar.sh
    echo " " >> setEnvar.sh
    echo "# Large netcdf file support: 0=Off, 1=On." >> setEnvar.sh
    echo "export WRFIO_NCD_LARGE_FILE_SUPPORT=1" >> setEnvar.sh
    ln setEnvar.sh $HOME/WRFhydromodel/wrf_hydro_nwm_public-5.2.0/trunk/NDHMS

    # Compile WRF-Hydro
    echo "Compiling WRF-Hydro..."
    cd $HOME/WRFhydromodel/wrf_hydro_nwm_public-5.2.0/trunk/NDHMS
    ./configure # Option 2
    ./compile_offline_NoahMP.sh setEnvar.sh

    # Copy .TBL and Executables
    echo "Copying .TBL and executables..."
    mkdir -p $HOME/WRFhydromodel/domain/NWM
    cp Run/*.TBL $HOME/WRFhydromodel/domain/NWM
    cp Run/wrf_hydro.exe $HOME/WRFhydromodel/domain/NWM

    # Download Croton_NY Test Case
    echo "Downloading Croton_NY Test Case..."
    cd $DOWNLOAD_DIR
    wget -c https://github.com/NCAR/wrf_hydro_nwm_public/releases/download/v5.2.0/croton_NY_training_example_v5.2.tar.gz

    # Extracting the tar.gz file
    tar -xvzf croton_NY_training_example_v5.2.tar.gz

    # Defining the source and destination paths for training examples
    SOURCE_DIR="example_case/NWM"
    DEST_DIR="$HOME/WRFhydromodel/domain/NWM"



    # Copying the FORCING Directory
    cp -r example_case/FORCING $DEST_DIR

    # Copying the DOMAIN Directory
    cp -r $SOURCE_DIR/DOMAIN $DEST_DIR

    # Copying the RESTART Directory
    cp -r $SOURCE_DIR/RESTART $DEST_DIR

    # Copying the nudgingTimeSliceObs Directory
    cp -r $SOURCE_DIR/nudgingTimeSliceObs $DEST_DIR

    # Copying the referenceSim Directory
    #cp -r $SOURCE_DIR/referenceSim $DEST_DIR

    # Copying the namelist.hrldas File
    cp $SOURCE_DIR/namelist.hrldas $DEST_DIR

    # Copying the hydro.namelist File
    cp $SOURCE_DIR/hydro.namelist $DEST_DIR

    # Verifying the copied files and directories
    ls -R $DEST_DIR


    # Navigating to the NWM Directory
    cd $HOME/WRFhydromodel/domain/NWM

    # Running WRF-Hydro with MPI: run programs in parallel using MPI (Message Passing Interface): This command runs the model in offline mode with Noah-MP based on the compiled executable.
    mpirun -np 2 ./wrf_hydro.exe

    Optional:
    # Running WRF-Hydro  while Error Checking
    # mpirun -np 2 ./wrf_hydro.exe > wrf_hydro_run.log 2>&1

    # Verifying the Output
    ls -lah HYDRO_RST*

    echo "Script finished."