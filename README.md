# TP2_setup
Instruction manual for installing TP2 with biogeochemistry to sigma2 HPC (Betsy etc.)

### Rquirements

Installation of TOPAZ ocean model system with biogeochemical modules requires the following model components and environment setups. 

#### Model components

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

#### Bash environment

Add the following line to `.bashrc`:

```bash
export LANG=en_US.UTF-8
export LC_ALL=en_US.utf8
```

#### HPC modules

Add the following lines to `.bashrc`: 

##### On Betzy

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

#### Environment variables for installation settings

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
