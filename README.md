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

After editing `.bashrc`, re-login or perform `source ~/.bashrc`. See [REDAME.betzy](https://github.com/nansencenter/NERSC-HYCOM-CICE/blob/develop/README.betzy) or [REDAME.fram](https://github.com/nansencenter/NERSC-HYCOM-CICE/blob/develop/README.fram) for the extra information.
