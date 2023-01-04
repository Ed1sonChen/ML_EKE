# Setup MOM6 environment on Linux system

This is a record to help people setup the environment with their own system and sucessfully reproduce the experiments of ML_EKE_linux (https://github.com/CrayLabs/NCAR_ML_EKE).

## Installation

Below are the steps to install the various components needed to reproduce the results in the paper for the ML-EKE parameterization with SmartSim

## My Environment
GPUs: Nvidia V100
python:3.7

### Smartsim Installation

```bash
pip install smartsim
conda install git-lfs
conda install cmake
```

### Build Smartsim CPU Install

```bash
bash -c "export CC=gcc"
bash -c "export CXX=g++"
bash -c "export NO_CHECKS=1"
pip install onnx==1.9.0
pip install tensorflow==2.5.2
smart build --device cpu --onnx
```

### Build Smartsim GPU Install

```bash
smart clean
smart build --device gpu --onnx
```
### Smartredis Install

```bash
pip install smartredis
```

### Build MOM6

#### Cloning the core repositories

```bash
git clone --recursive https://github.com/NOAA-GFDL/MOM6-examples.git MOM6-examples
```


#### Build tools and compilation templates
MOM6 relies on tools that are part of the Flexible Runtime Environment (FRE) to help with the build process. Those tools are provided in the NOAA-GFDL/mkmf repository which you should find in MOM6-examples/src/mkmf/.

The workflow we describe in these wiki pages generally compiles the model in a user created directory MOM6-examples/build/:

```bash
cd MOM6-examples
mkdir build
```

### Compiling the models

#### Modules need to be installed 

gfortran
netcdf4
netcdf-fortran
mpich

```bash
conda config --set channel_priority strict
conda install -c conda-forge gfortran -y
conda install -c conda-forge netcdf4 -y
conda install -c conda-forge netcdf-fortran -y
conda install -c conda-forge mpich -y
```

#### Compiling MOM6 in ocean-only mode

```bash
mkdir -p build/ocean_only/
(cd build/ocean_only/; rm -f path_names; \
../../src/mkmf/bin/list_paths -l ./ ../../src/MOM6/{config_src/infra/FMS1,config_src/memory/dynamic_symmetric,config_src/drivers/solo_driver,config_src/external,src/{*,*/*}}/ ; \
../../src/mkmf/bin/mkmf -t ../../src/mkmf/templates/osx-gcc10.mk -o '-I../fms' -p MOM6 -l '-L../fms -lfms' path_names)
```

On your platform, replace ```osx-gcc10.mk ``` with the appropriate template file.

Finally, compile the MOM6 ocean-only model with:
```bash
(cd build/ocean_only/; source ../env; make REPRO=1 MOM6 -j)
```
If successful you should now have an ocean executable: ```build/ocean_only/MOM6 ```.

### Running a test case

```bash
cd ocean_only/double_gyre/
mkdir -p RESTART
mpirun -n 8 ../../build/ocean_only/MOM6
```





