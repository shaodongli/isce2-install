# ISCE2 installation guide on MacOS with Apple Silicon

This guide provides instructions to install ISCE2 with Anaconda/Miniconda on a Linux/MacOS machine. 
**NOTE**: this is not the **official** installation guide. It only serves to help users to install ISCE2 on some common and most recent platforms. Please check the [ISCE2](https://github.com/isce-framework/isce2) page for official guides and tutorials.  

# MacOSX with Anaconda3 and homebrew: Apple Silicon 

(Tested on macOS Sequoia 15.2. This is the recommended method for MacOS - all packages are pre-compiled. However, after a major MacOS upgrade, e.g., from 13 to 14, a re-installation of Xcode Command Line Tools, conda, homebrew is recommended.)    

1. Install Command Line Tools, Conda and gcc/g++/gfortran Compiler

Install Command Line Tools, (NOTE as 11/7/24: if you used Settings->Software Updates option, the CLT has a bug. Remove it and reinstall!!!) 

      sudo rm -rf /Library/Developer/CommandLineTools/
      xcode-select --install 

then follow the popup window to install. 

(I encountered a segmentation fault during the step `stripmapApp.py stripmapApp.xml --start=startup --end=preprocess` before updating the CLT)

Install an osx-arm64 build of Anaconda3

Install Homebrew (the pkg installer is the easiest method, download from [Homebrew Releases](https://github.com/Homebrew/brew/releases/)) -- find the most recent Homebrew-n.n.n.pkg file. For Apple Silions (osx-arm64), brew is installed to ``/opt/homebrew``. 

        export PATH="/opt/homebrew/bin:$PATH"

and then install gfortran (current version GCC 14.2.0)

        brew install gfortran
        /opt/homebrew/bin/gfortran --version # check 

If you need mdx (slc viewing software), install openmotif here (osx-arm64 version currently not available from conda)

        brew install openmotif

Also install [XQuartz](https://www.xquartz.org/). 
        
2. Prepare a virtual environment with conda or mamba 

       conda create -n isce2
       conda activate isce2

The following steps will install isce2 to $CONDA_PREFIX. 

       echo $CONDA_PREFIX 

3. Install required packages

       conda install python=3.12 opencv jupyter matplotlib sardem boto3 tqdm git cmake cython gdal h5py libgdal pytest numpy=1.26.4 fftw scipy pybind11 shapely
       
I try to avoid managing a conda environment with both `pip` and `conda` as much as possible.
`XmlDumper.py` is not happy with `numpy>=2`, but I have not tested other `numpy` packages than `numpy=1.26.4`.
                     
4. Compile and install isce2

Make a link to make the installation path easier (``-DPYTHON_MODULE_DIR`` not longer need)

       ln -sf `python3 -c 'import site; print(site.getsitepackages()[0])'` $CONDA_PREFIX/packages

Download ISCE2 from github

        git clone https://github.com/isce-framework/isce2.git

Compile ISCE2 with cmake, 

        cd isce2
        mkdir build && cd build
        cmake .. -DCMAKE_INSTALL_PREFIX=$CONDA_PREFIX \
          -DCMAKE_PREFIX_PATH=${CONDA_PREFIX} \
          -DCMAKE_C_COMPILER="/opt/homebrew/bin/gcc-14" \
          -DCMAKE_CXX_COMPILER="/opt/homebrew/bin/g++-14" \
          -DCMAKE_Fortran_COMPILER="/opt/homebrew/bin/gfortran-14"
        make -j # to use multiple threads
        make install       

We use gcc from homebrew instead of Apple Clang because of some compatibility issue (some source codes need to be updated). 

5. Config and run isce2

If you follow the above steps, ISCE2 packages are installed to $CONDA_PREFIX/packages/isce2. You will only need to add the path to stack apps, 

        export ISCE_HOME="$CONDA_PREFIX/packages/isce"
        export PATH="$ISCE_HOME/applications:$PATH"

If you have installed ISCE2 to a custom directory, e.g., ``$HOME/apps/isce2``, with ``-DCMAKE_INSTALL_PREFIX=$HOME/apps/isce2`` cmake option, you need to 

        export ISCE_INSTALL_ROOT="$HOME/apps/isce2"
        export ISCE_HOME="$ISCE_INSTALL_ROOT/packages/isce"
        export PATH="$ISCE_HOME/applications:$ISCE_INSTALL_ROOT/bin:$PATH"
        export PYTHONPATH="$ISCE_INSTALL_ROOT/packages:$PYTHONPATH"

You may try the following to check whether ISCE2 has been properly installed, 

        python3 -c "import isce"

To use mdx, you will need [XQuartz](https://www.xquartz.org/). 

        mdx.py xxxxx.slc 
        # show the slc picture (.xml description file needed)

6. Register a python nobebook kernel (optional)

        python -m ipykernel install --user --name isce2

7. Install other packages used in EarthScope 2024 ISCE+ short course (optional)

        conda install -c conda-forge asf_search -y
        conda install xarray -y
        conda install conda-forge::awscli -y
        conda install -c conda-forge sentineleof -y
        conda install conda-forge::rasterio -y

        # for ARIA-tools, see https://github.com/aria-tools/ARIA-tools
        conda install arm_pyart joblib netcdf4 parallel tile_mate
        git clone https://github.com/aria-tools/ARIA-tools.git
        cd ARIA-tools
        python -m pip install -e .

        
    
Enjoy!

