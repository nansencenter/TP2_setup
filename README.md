# TP2_setup
Instruction manual for installing TP2 with biogeochemistry to sigma2 HPC (Betsy etc.) This manual is developed based on 
[Shuang's TP2 setup note](https://docs.google.com/document/d/1z53p0V0knFjaSZTXhO1WUQvG1OMDk7nzLdp_zZKGwJk/edit?tab=t.0)

## TOCs: Table of Contents
- [Rquirements](#requirements)
  - [*Model components*](#model-components)
  - [*Bash environment*](#bash-environment)

### Rquirements

Installation of TOPAZ ocean model system with biogeochemical modules requires the following model components and environment setups. 

#### *Model components*

Each component is downloaded from its associated git repository:

1. [NERSC-HYCOM-CICE (develop branch)](https://github.com/nansencenter/NERSC-HYCOM-CICE/tree/develop)
   NERSC-HYCOM-CICE is an ocean-sea ice-biogeochemistry coupled model framework developed at NERSC.
2. [ECOSMO_operational (main branch)](https://github.com/nansencenter/nersc/blob/main/ecosmo/ecosmo_operational.F90)
   ECOSMO_operational is a biogeochemical modeling module optimized for operational tasks at NERSC. EOCSMO_operational package (nersc) is downloaded as a part of FABM package, but we use the latest version from NERSC git repository.
3. [GOTM](https://github.com/gotm-model/code)
   GOTM is a 1D column ocean model. In this installation we use a light module of GOTM installed as a part of FABM package.
5. [FABM](https://github.com/fabm-model/fabm)
   FABM is a Fortran 2003 programming framework (a coupler) for biogeochemical models of marine and freshwater systems. 
6. [ERSEM](https://github.com/pmlmodelling/ersem)
   ERSEM is a marine biogeochemical and ecosystem model. It describes the cycling of carbon, nitrogen, phosphorus, silicon, oxygen and iron through the lower trophic level pelagic and benthic ecosystems. In this installation we use a carbon chenistry module of ERSEM.
7. NERSC Python library (downloaded together with NERSC-HYCOM-CICE package).

#### *Bash environment*

Add the following line to `.bashrc`:

```bash
export LANG=en_US.UTF-8
export LC_ALL=en_US.utf8
```

#### *HPC modules*

Add the following lines to `.bashrc`: 

##### *On Betzy*

```bash
ml purge
ml load CMake/3.23.1-GCCcore-11.3.0
ml load ESMF/8.3.0-iomkl-2022a
ml load FFTW/3.3.10-GCC-11.3.0
ml load UDUNITS/2.2.28-GCCcore-11.3.0
ml load Python/3.10.4-GCCcore-11.3.0
ml load GSL/2.7-intel-compilers-2022.1.0
```

After editing `.bashrc`, re-login or perform `source ~/.bashrc`. See [REDAME.betzy](https://github.com/nansencenter/NERSC-HYCOM-CICE/blob/develop/README.betzy) for  extra information.

### Installation

In this instruction, we use the following directory tree structure for hycom (version 2.2) installation on Betzy:
```bash
  /cluster
  ├── home
  │   └── ${USERNAME}                            # HOME
  │       ├── ${MODELNAME}                       # HOME_HYCOM (HYCOM-CICE home)
  │       │   └── ${HYCOM_REPO}
  │       │       ├── pythonlibs                 # LIB_PYTHON (NERSC HYCOM Python libs)
  │       │       ├── bin                        # BIN_HYCOM (NERSC HYCOM utility scripts)
  │       │       └── hycom
  │       │           ├── RELO
  │       │           │   └── src_2.2.98ZA-07Tsig0-i-sm-sse_relo_mpi # SRC_HYCOM (HYCOM source files)
  │       │           └── MSCPROGS
  │       │               ├── bin                # BIN_HYCOM_MSCPROGS (MSCPROGS bin folder)
  │       │               └── lib/libhycnersc.a  # LIB_HYCNERSC (MSCPROGS Nersclib library)
  │       ├── FABM                               # HOME_FABM (FABM/ECOSMO home)
  │       └── local/fabm/hycom/lib64/libfabm.a   # LIB_FABM (FABM library)    
  │
  └── work
      └── users
          └── ${USERNAME}
              └── ${MODELNAME}                   # WORK_HYCOM (HYCOM-CICE work)
                  └── ${CONFIGNAME}              # WORK_HYCOM_CNF (configuration folder)
                      ├── REGION.src
                      ├── expt_${NEWEXPERIMENT}  # WORK_HYCOM_EXP (experiment folder)
                      │   ├── EXPT.src
                      │   ├── SCRATCH            # WORK_HYCOM_SCR (SCRATCH folder)
                      │   ├── data               # WORK_HYCOM_DAT (data/restart files folder)
                      │   └── build
                      │       └── src_2.2.98ZA-07Tsig0-i-sm-sse_relo_mpi # HYCOM source files (copy from SRC_HYCOM)
                      │           └── hycom_cice # hycom executable
                      ├── nest
                      │   └── ${IEXPT}           # WORK_HYCOM_NST (nesting files folder)
                      ├── relax
                      │   └── ${IEXPT}           # WORK_HYCOM_RLX (relaxation files folder)
                      └── force
                          └── synoptic
                              └── ${IEXPT}       # WORK_HYCOM_FRC (forcing files folder)
```

#### *Environment variables for installation settings*

In order to make installation processes available through bash scripts, we set some key environment variables first:
```bash
export CONFIGNAME="TP2a0.10"   # configuration name 
export NEWEXPERIMENT=04.2      # new experiment number
export T="04"                  # topography version number
export IVERSN=22               # hycom version number x10
export MXBLCKS=9               # maximal number of ice blocks relates to MPI cores distribution
export ICECLIM=0               # prepare initial ice file from climatology (1) or not (otherwise)
export INITFLG=""              # start physics from climatology "--init" or from restart ""
export RSTRFQ=7                # restart file dumping frequency [day]
export MSLPRF=0                # msl pressure forcing flag (0=F,1=T)
export NRDFLG=0                # options for the radiation heat fluxes(0,1,...,7). Only to hycom 2.2 
export COMPILE_BIOMODEL="yes"  # turn on/off ("yes/no") BGC (FABM-ECOSMO) module
export NTRACR=1                # BGC on/off: 0-physics only; 1-biology restart; -1-biology initialized with climatology
export NMPI=504                # number of MPI tiles
```
for TP2 configuration with ECOSMO (BGC module) on Betzy. To setup hycom without ECOSMO choose:
```bash
export COMPILE_BIOMODEL="no"   # turn on/off ("yes/no") BGC (FABM-ECOSMO) module
export NTRACR=0                # BGC on/off: 0-physics only; 1-biology restart; -1-biology initialized with climatology
```
Note `NEWEXPERIMENT` is user defined variable with format `NN.M` that defines experiment folder name (see directory tree below), where `NN` is experiment folder ID (two degits integer) and `M` is its version ID  (sigle degit integer). 

Define HYCOM-CICE repository as
```bash
export HYCOM_REPO=NERSC-HYCOM-CICE                               # HYCOM-CICE repository name 
```
for hindcast/forecast run or as
```bash
export HYCOM_REPO=NERSC-HYCOM-CICE_BIORANv2                      # HYCOM-CICE repository name 
```
for BGC reanaysis run.

Next, we define `MODELNAME`. Choice of `MODELNAME` is up to user and for example, you can use
```bash
export MODELNAME='TP2_HINDCAST'                                              # common model folder name under HOME/WORK
```
or 
```bash
export MODELNAME='TP2_REANALYSIS'                                            # common model folder name under HOME/WORK
```

Another example is to use shortname of model configuration TP2a0.10:
```bash
set -u
export SHRTNAME="$(echo "$CONFIGNAME" | sed -E 's/[a-z].*$//')"              # eg. TP2a0.10 > TP2
export MODELNAME=$SHRTNAME                                                   # common model folder name under HOME/WORK
```

After setting bash variables, skelton of the directory tree can be initialized by bash scripts:

```bash
# setup directory tree
set -u

## define variables
export USERNAME=$(whoami)                                                      # your username
export IEXPT=$(echo "$NEWEXPERIMENT" | sed 's/\.//')                           # eg. "03.0" > "030" 
export HOME_HYCOM=/cluster/home/${USERNAME}/${MODELNAME}                       # HYCOM-CICE home directory
export HOME_FABM=/cluster/home/${USERNAME}/FABM                                # FABM(ECOSMO) home directory
export WORK_HYCOM=/cluster/work/users/${USERNAME}/${MODELNAME}                 # HYCOM-CICE work directory
export LIB_PYTHON=$HOME_HYCOM/$HYCOM_REPO/pythonlibs                           # NERSC hycom python libraries directory
export LIB_HYCNERSC=$HOME_HYCOM/$HYCOM_REPO/hycom/MSCPROGS/lib/libhycnersc.a   # Hycomlib library
export LIB_FABM=/cluster/home/${USERNAME}/local/fabm/hycom/lib64/libfabm.a     # FABM library
export BIN_HYCOM=$HOME_HYCOM/$HYCOM_REPO/bin                                   # hycom utility bin diectory
export BIN_HYCOM_MSCPROGS=$HOME_HYCOM/$HYCOM_REPO/hycom/MSCPROGS/bin           # hycom MSCPROGS bin diectory
export PATH="$BIN_HYCOM_MSCPROGS:$BIN_HYCOM:$HOME_HYCOM/$HYCOM_REPO/bin:$PATH" # add utilty bins to PATH

## create HOME_ and WORK_ directories
mkdir -p $HOME_HYCOM
mkdir -p $HOME_FABM        
mkdir -p $WORK_HYCOM
```
where `HOME_##` specifies a folder for storing model source files (*home*) and `WORK_##` specifies a folder for model configuration and execution (*build*).

#### *Cloning the model components*

Now you are ready to clone hycom under `HOME_HYCOM`:
```bash
set -u
# clone HYCOM-CICE
cd $HOME_HYCOM
git clone https://github.com/nansencenter/${HYCOM_REPO}.git

# switch to develop branch
cd $HOME_HYCOM/${HYCOM_REPO}
git fetch --all
git checkout develop
```
and to clone biogeochemical modules under `HOME_FABM`:
```bash
set -u
cd $HOME_FABM
# clone FABM
git clone https://github.com/fabm-model/fabm.git         # FABM

# clone ERSEM
git clone https://github.com/pmlmodelling/ersem.git      # ERSEM

# clone ECOSMO_operational
git clone git@github.com:nansencenter/nersc.git          # ECOSMO
#cp -r /cluster/projects/nn9481k/caglar/ECOSMO_code/nersc # ECOSMO
```

Nersc Python libraries can be installed by
```bash
pip install --user scipy
pip install --user cfunits
pip install --user netCDF4
pip install --user netcdftime
pip install --user numpy
pip install --user f90nml
pip install --user pyproj
```
and
```bash
pip install --user $LIB_PYTHON/modeltools
pip install --user $LIB_PYTHON/modelgrid
pip install --user $LIB_PYTHON/gridxsec
pip install --user $LIB_PYTHON/abfile
```
for the first installation and 
```bash
pip install --user --upgrade $LIB_PYTHON/modeltools
pip install --user --upgrade $LIB_PYTHON/modelgrid
pip install --user --upgrade $LIB_PYTHON/gridxsec
pip install --user --upgrade $LIB_PYTHON/abfile
```
for updating the exsisting libraries.

### Createing a new experiment

Before compilation of hycom executable, we need to create a new experiment folder ``.

#### *Copy configuration to a work directory*

First, copy hycom configuration and create symlink of hycom bin. For TP2, perform:
```bash
set -u
cd $WORK_HYCOM
cp -r $HOME_HYCOM/${HYCOM_REPO}/$CONFIGNAME .
cd $WORK_HYCOM/$CONFIGNAME
[ -f bin ] && rm bin
ln -sf $HOME_HYCOM/${HYCOM_REPO}/bin .
```

#### *Adjust experiment settings in REGION.src*

Next, you need to prepare `REGION.src` for your experimental settings BEFORE making a new *experiment*. 
Normally, this step is done manually, but here is a bash script to achieve the step at command line:
```bash
set -u
cd $WORK_HYCOM/$CONFIGNAME
cp $HOME_HYCOM/${HYCOM_REPO}/input/REGION.src .
sed -i "/^export R=/c export R=$CONFIGNAME" REGION.src
sed -i "/^export NHCROOT=/c export NHCROOT=$HOME_HYCOM/${HYCOM_REPO}" REGION.src
```
where `R` is model configuration name and `NHCROOT` is where HYCOM-CICE source files are downloaded.

#### *Create a new experiment*

Now you create a new *experiment* from the existing *experiment* (01.0) under hycom work/configuration directory (which is the case if you clone HYCOM-CICE from git repository).
```bash
set -u
cd $WORK_HYCOM/$CONFIGNAME
bin/expt_new.sh 01.0 $NEWEXPERIMENT
```

After a new *experiment* folder `expt_$NEWEXPERIMENT` is created, some adjustments of `blkdat.input` and `EXPT.src` under the folder are required.

#### *Copy and edit hycom_opt* ⚠️**NEW**

```bash
cd $WORK_HYCOM/$CONFIGNAME/expt_$NEWEXPERIMENT
cp $HOME_HYCOM/$HYCOM_REPO/TP5a0.06/expt_01.0/hycom_opt .
sed -i 's/sssrmx_scalar=99\./sssrmx_scalar=.5/' hycom_opt
```

#### *Modify experiment settings in blkdat.input*

Experiment settings defined in the original `blkdat.input` are needed to be adjusted for specific hycom settings. 
You can copy `blkdat.input` from existing TP2 configuration with nesting boundary
```bash
BLKDATIN=/nird/datalake/NS9481K/shuang/TP2_setup/exp02.6_seaclim_ref/blkdat.input # TP2 with nesting boundary
```
or with climatology boundary
```bash
BLKDATIN=/nird/datalake/NS9481K/shuang/TP2_setup/exp02.6_seaclim_ref/blkdat.input_clim # TP2 with climatology boundary
```
then run
```bash
set -u
cd $WORK_HYCOM/$CONFIGNAME/expt_$NEWEXPERIMENT
mv blkdat.input blkdat.input_backup
cp $BLKDATIN blkdat.input
```

After importing new `blkdat.input`, you need to modify `iexpt`, `ntracr` and `rstefq` in `blkdat.input` depending on your experiment settings manually with file editor or run scripts below at command line:
```bash
set -u
vars=("$IEXPT" "$RSTRFQ" "$NTRACR")
keys=( "iexpt " "rstrfq"  "ntracr")
```
first and run:
```bash
set -u

cd $WORK_HYCOM/$CONFIGNAME/expt_$NEWEXPERIMENT
filename='blkdat.input'

# Define a dictionary (an associative array) of keyword and replacement.
declare -A replacements 
for i in "${!keys[@]}"; do
    replacements["${keys[$i]}"]="${vars[$i]}"
done

tempfile=$(mktemp)
while IFS= read -r line; do
    for keyword in "${!replacements[@]}"; do
        replacement="${replacements[$keyword]}"
        if [[ "$line" == *"$keyword"* ]]; then
            line=$(echo "$line" | sed -E "s/^([[:space:]]*)(-?[0-9]+)/\1$replacement/")
            break
        fi
    done
    echo "$line" >> "$tempfile"
done < "$filename"
mv "$tempfile" "$filename"

for key in "${keys[@]}"; do
    grep "$key" $filename
done
```

With hycom 2.2, we create new `blkdat.input` based on `blkdat.input_H2.298` by first setting environment variables:
```bash
set -u

cd $WORK_HYCOM/$CONFIGNAME/expt_$NEWEXPERIMENT

export VELDF4=-2               # diffusion velocity (m/s) for biharmonic momentum dissip. (use < 0 for reading exterbal file)
export THKDF4=-2               # diffusion velocity (m/s) for biharmonic thickness diffus. (use < 0 for reading exterbal file)
export TICEGR=2                # ENLN: temp. grad. inside ice (deg/m); =0 use surtmp
export MSLPRF=0                # msl pressure forcing flag (0=F,1=T)

vars=("$IEXPT" "$RSTRFQ" "$NTRACR" "$VELDF4" "$THKDF4" "$TICEGR" "$MSLPRF")
keys=( "iexpt " "rstrfq"  "ntracr"  "veldf4"  "thkdf4"  "ticegr"  "mslprf")

cp blkdat.input_H2.298 blkdat.input
```
then run:
```bash
set -u

cd $WORK_HYCOM/$CONFIGNAME/expt_$NEWEXPERIMENT
filename='blkdat.input'

# Define a dictionary (an associative array) of keyword and replacement.
declare -A replacements 
for i in "${!keys[@]}"; do
    replacements["${keys[$i]}"]="${vars[$i]}"
done

tempfile=$(mktemp)
while IFS= read -r line; do
    for keyword in "${!replacements[@]}"; do
        replacement="${replacements[$keyword]}"
        if [[ "$line" == *"$keyword"* ]]; then
            line=$(echo "$line" | sed -E "s/^([[:space:]]*)(-?[0-9]+)/\1$replacement/")
            break
        fi
    done
    echo "$line" >> "$tempfile"
done < "$filename"
mv "$tempfile" "$filename"

for key in "${keys[@]}"; do
    grep "$key" $filename
done
```
⚠️**Note**: This script is confirmed only with keywords in the above sample script. When you add other keyword to `replacements`, make sure first the string replacement is working as intended. 

#### *Modify experiment settings in EXPT.src*

Under `$WORK_HYCOM/$CONFIGNAME/topo` folder, you can find model topography data files `depth_TP2a0.10_01.[a,b]` where the last two digits represnt version number of the topography files. Here, we choose version `04` by setting `T="04"` in `EXPT.src`. Newly added variable `COMPILE_BIOMODEL` defines if HYCOM-CICE is coupled with ECOSMO. When `COMPILE_BIOMODEL="yes"`, the coupler is turned on and is turned off otherwise (eg, `COMPILE_BIOMODEL="no"`). The maximal number of ice blocks `MXBLCKS` relates to the MPI cores distribution, and could be recommened by the error message in a model run.
Normally, this step is done manually, but here is a bash script to achieve the step at command line:
```bash
set -u
cd $WORK_HYCOM/$CONFIGNAME/expt_$NEWEXPERIMENT
#echo ""                                                >> EXPT.src
#echo "# add new parameters for FABM and ICE"           >> EXPT.src
#echo "export MXBLCKS=${MXBLCKS}"                       >> EXPT.src # maximal number of ice blocks 
#echo "export COMPILE_BIOMODEL=\"${COMPILE_BIOMODEL}\"" >> EXPT.src # FABM coupler ON ("yes") or OFF ("no")
sed -i "/^X=/c X=\"$NEWEXPERIMENT\"" EXPT.src                       # X based on dir name
sed -i "/^E=/c E=\"$IEXPT\"" EXPT.src                               # E is X without "."
sed -i "/^T=/c T=\"$T\"" EXPT.src                                   # topography version
sed -i "/^export NMPI=/c export NMPI=$NMPI" EXPT.src
sed -i "/^export MXBLCKS=/c export MXBLCKS=$MXBLCKS" EXPT.src
sed -i "/^export COMPILE_BIOMODEL=/c export COMPILE_BIOMODEL=\"${COMPILE_BIOMODEL}\"" EXPT.src
```

- this command should be part of expt_new.sh with COMPILE_BIOMODEL, T as command line arguments.
- if you get error message about ice blocks exceed maximum, then change this number to the once recommend in the error message.

### Compiling hycom executable (hycom_cice)

Compilation of HYCOM_CICE with biogeochemical modules requires 2 pre-compiled libraries `libhycnersc.a` and `libfabm.a`.

#### *Compile libhycnersc.a/MSCPROGS*

First, setup proper Makefile include file `make.inc` for your environment:

```bash
set -u
cd $HOME_HYCOM/${HYCOM_REPO}/hycom/MSCPROGS/src/Make.Inc

compiler=ifort # FORTRAN compiler (eg. gfortran, gnu)
hostname=$(head -n 1 /etc/motd | awk -F' ' '{split($3, parts, "."); print parts[1]}') # extract hostname from motd
echo $hostname
[ -L make.inc ] && rm make.inc
ln -sf make.$hostname.$compiler make.inc
ls -l make.inc
```
⚠️**Note**: This script is confirmed only on sigma2 Fram and Betzy.

For example on Betzy, this script creates symlink `make.inc`:

```bash
make.inc -> make.betzy.ifort
```

If you set `HYCOM_REPO=NERSC-HYCOM-CICE_BIORANv2`, add the following patch before compilation (TODO: fix them on git repository):
```bash
set -u
mkdir -p $HOME_HYCOM/${HYCOM_REPO}/hycom/MSCPROGS/src/Average/TMP     # debug!
mkdir -p $HOME_HYCOM/${HYCOM_REPO}/hycom/MSCPROGS/src/Hycom_mean/TMP  # debug!
mkdir -p $HOME_HYCOM/${HYCOM_REPO}/hycom/MSCPROGS/src/Perturb_Forcing-2.2.98_TP5/TMP # debug!
mkdir -p $HOME_HYCOM/${HYCOM_REPO}/hycom/MSCPROGS/src/Perturb_Parameter/TMP # debug!
```

Now, compile MSCPROGS:
```bash
set -u
cd $HOME_HYCOM/${HYCOM_REPO}/hycom/MSCPROGS/src
gmake clean
gmake all
```

Once compilation successed, install executables under `MSCPROGS/bin` and `MSCPROGS/bin_setup`:

```bash
gmake install
```

After the compilation/installation, `libhycnersc.a` can be found under
```bash
$HOME_HYCOM/$HYCOM_REPO/hycom/MSCPROGS/lib
```

⚠️**Note**: The installation should be repeated everytime when HPC modules are renewed.

For further detail, see [Procedure for compiling/installing MSCPROGS](https://github.com/nansencenter/NERSC-HYCOM-CICE/tree/develop/hycom/MSCPROGS)

#### *Compile libfabm.a/FABM*

For clean install, remove `build` folder:
```bash
set -u
cd $HOME_FABM
[ -d build ] && rm -rf build
```
then compile fabm:
```bash
set -u
cd $HOME_FABM
mkdir build; cd build
cmake $HOME_FABM/fabm -DFABM_HOST=hycom -DCMAKE_Fortran_COMPILER=ifort -DFABM_INSTITUTES="ersem;nersc;gotm" -DFABM_NERSC_BASE=$HOME_FABM/nersc -DFABM_ERSEM_BASE=$HOME_FABM/ersem
make install
```

After the compilation, `libfabm.a` can be found under
```bash
${HOME}/local/fabm/hycom/lib64
```

#### *Compile HYCOM-CICE*

⚠️**Note**: BGC module should be turned on in `blkdat.input` for physics-BGC coupled model compilation by setting bash environment variable `NTRACR`.

Before the compilation, we need to copy FABM configuration files, `fabm.yaml` and `hycom_fabm.nml` and CICE namelist `ice_in`, to the `experiment` folder. In this installation manual, we use experiment setups for TP2 hindcast run:

```bash
set -u
cd $WORK_HYCOM/$CONFIGNAME/expt_$NEWEXPERIMENT
cp  /nird/datalake/NS9481K/shuang/TP2_setup/exp02.6_seaclim_ref/fabm.yaml .
cp  /nird/datalake/NS9481K/shuang/TP2_setup/exp02.6_seaclim_ref/hycom_fabm.nml .
cp  /nird/datalake/NS9481K/shuang/TP2_setup/exp02.6_seaclim_ref/ice_in .
```

For clean install, remove `build` folder:
```bash
set -u
cd $WORK_HYCOM/$CONFIGNAME/expt_$NEWEXPERIMENT
[ -d build ] && rm -rf build
```
then compile HYCOM-CICE:
```bash
set -u
cd $WORK_HYCOM/$CONFIGNAME/expt_$NEWEXPERIMENT
ln -sf $HOME_HYCOM/${HYCOM_REPO}/bin/compile_model.sh .    # temporal solution
ln -sf $HOME_HYCOM/${HYCOM_REPO}/bin/common_functions.sh . # temporal solution
bash compile_model.sh ifort -u
```

Once compilation finished, HYCOM-CICE executable `hycom_cice` is created under 
```bash
$WORK_HYCOM/$CONFIGNAME/expt_$NEWEXPERIMENT/build/src_2.2.98ZA-07Tsig0-i-sm-sse_relo_mpi`
```

For extra details, see [README.betzy](https://github.com/nansencenter/NERSC-HYCOM-CICE/blob/develop/README.betzy) or [README.fram](https://github.com/nansencenter/NERSC-HYCOM-CICE/blob/develop/README.fram).

#### *Compile hycom_ALL*

Before preparing external files to run hycom, hycom utilities `hycom_ALL` should be compiled. First, setup proper Makefile include file `Make_all.src` for your environment. Normally, this step is done manually, but here is a bash script to achieve the step at command line:
```bash
set -u
cd $HOME_HYCOM/${HYCOM_REPO}/hycom/hycom_ALL/hycom_2.2.72_ALL
compiler=intelIFC
if [ -f config/${compiler}_setup ]; then
   sed -i "/^setenv ARCH /c setenv ARCH $compiler" Make_all.src
else
   echo "Can not find setup file ${compiler}_setup, EXIT"
fi
```
⚠️**Note**: This script is confirmed only on sigma2 Fram and Betzy.

Then compile:
```bash
csh ./Make_clean.com
csh ./Make_all.com
csh ./Make_ncdf.com
```
Note compilation of `Make_ncdf.com` would be stuck due to error for the moment.
For further detail, see [Procedure for compiling programs in hycom_2.2.72_ALL](https://github.com/nansencenter/NERSC-HYCOM-CICE/blob/develop/hycom/hycom_ALL/README.md)

⚠️**Note [2026/01/14]**: These files failed to be compiled in ```Make_all.com```
```bash
hycom_profile_hybgen.f
hycom_profile_hybgen+.f
```
