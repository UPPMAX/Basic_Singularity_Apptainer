# NAMD AMD GPUs ([Kebnekaise](https://www.hpc2n.umu.se/resources/hardware/kebnekaise))

Although some parallel software could experience some performance overhead by using
containers instead of the compiled versions, sometimes this is still a good alternative.
One case is NAMD3 a software for Molecular Dynamics when it needs to run on AMD GPUs. 

## Setup 

[AMD](https://www.amd.com/en/developer/resources/infinity-hub/namd3.html) offers containers for this NAMD version
that can run straightforwardly on AMD GPUs, for instance, MI100s. On the machine you 
are targeting:

``` apptainer
apptainer pull docker://amdih/namd3:3.0a9

apptainer shell namd3_3.0a9.sif
```

This software requires that you accept the terms of a license. The license together with 
examples and benchmarks can be found at [here](https://www.ks.uiuc.edu/Research/namd/benchmarks/).
One can use the following batch script to run this software. 

``` slurm
#!/bin/bash
#SBATCH -A Project_ID
#SBATCH -t 20:17:00
#SBATCH -c 14
#SBATCH -o output_%j.out          # output file
#SBATCH -e error_%j.err           # error messages
#SBATCH --gpus-per-node=mi100:1   # using MI100 GPU cards
 
ml purge > /dev/null 2>&1
apptainer run /your-project-folder/NAMD3_ROCM/namd3_3.0a9.sif namd3 apoa1_hip.namd +p1 +setcpuaffinity  +devices 0 > output.log
```

In the *output.log* file you can see that NAMD3 is using the allocated resources:

``` namd
Charm++> Running on 1 hosts (1 sockets x 14 cores x 1 PUs = 14-way SMP)
Charm++> cpu topology info is gathered in 0.002 seconds.
Pe 0 physical rank 0 binding to CUDA device 0 on b-cn1612.hpc2n.umu.se: ''  Mem: 32752MB  Rev: 9.0  PCI: 0:27:0
Info: NAMD 3.0alpha9 for Linux-x86_64-multicore-HIP
Info: 
Info: Please visit http://www.ks.uiuc.edu/Research/namd/
Info: for updates, documentation, and support information.
Info: 
Info: Please cite Phillips et al., J. Chem. Phys. 153:044130 (2020) doi:10.1063/5.0014475
Info: in all publications reporting results obtained with NAMD.
```
