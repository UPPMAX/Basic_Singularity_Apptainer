# Alphafold3 ([Kebnekaise](https://www.hpc2n.umu.se/resources/hardware/kebnekaise))

Other times, installation of some software are not provided besides docker containers
(which are not supported in Swedish HPC centers) as in the case of Alphafold, a 
software for protein structure prediction. 

## Setup 

On the [Alphafold3](https://github.com/google-deepmind/alphafold3/tree/main) repository you can find the code that you can clone. 
In the root directory of the clonned repository you can place the following definition file (taken from issue #23
in this repository): 

``` AF3.def
Bootstrap: docker
From: nvidia/cuda:12.6.0-base-ubuntu22.04
Stage: spython-base

%files
. /app/alphafold
%post
# Copyright 2024 DeepMind Technologies Limited
#
# AlphaFold 3 source code is licensed under CC BY-NC-SA 4.0. To view a copy of
# this license, visit https://creativecommons.org/licenses/by-nc-sa/4.0/
#
# To request access to the AlphaFold 3 model parameters, follow the process set
# out at https://github.com/google-deepmind/alphafold3. You may only use these
# if received directly from Google. Use is subject to terms of use available at
# https://github.com/google-deepmind/alphafold3/blob/main/WEIGHTS_TERMS_OF_USE.md


# Some RUN statements are combined together to make Docker build run faster.
# Get latest package listing, install software-properties-common, git and wget.
# git is required for pyproject.toml toolchain's use of CMakeLists.txt.
apt update --quiet \
&& apt install --yes --quiet software-properties-common \
&& apt install --yes --quiet git wget

# Get apt repository of specific Python versions. Then install Python. Tell APT
# this isn't an interactive TTY to avoid timezone prompt when installing.
add-apt-repository ppa:deadsnakes/ppa \
&& DEBIAN_FRONTEND=noninteractive apt install --yes --quiet python3.11 python3-pip python3.11-venv python3.11-dev
python3.11 -m venv /alphafold3_venv
PATH="/hmmer/bin:/alphafold3_venv/bin:$PATH"

# Install HMMER. Do so before copying the source code, so that docker can cache
# the image layer containing HMMER.
mkdir /hmmer_build /hmmer ; \
wget http://eddylab.org/software/hmmer/hmmer-3.4.tar.gz --directory-prefix /hmmer_build ; \
(cd /hmmer_build && tar zxf hmmer-3.4.tar.gz && rm hmmer-3.4.tar.gz) ; \
(cd /hmmer_build/hmmer-3.4 && ./configure --prefix /hmmer) ; \
(cd /hmmer_build/hmmer-3.4 && make -j8) ; \
(cd /hmmer_build/hmmer-3.4 && make install) ; \
(cd /hmmer_build/hmmer-3.4/easel && make install) ; \
rm -R /hmmer_build

# Copy the AlphaFold 3 source code from the local machine to the container and
# set the working directory to there.
mkdir -p /app/alphafold
cd /app/alphafold

# Install the Python dependencies AlphaFold 3 needs.
pip3 install -r dev-requirements.txt
pip3 install --no-deps .
# Build chemical components database (this binary was installed by pip).
build_data

# To work around a known XLA issue causing the compilation time to greatly
# increase, the following environment variable setting XLA flags must be enabled
# when running AlphaFold 3:
XLA_FLAGS="--xla_gpu_enable_triton_gemm=false"
# Memory settings used for folding up to 5,120 tokens on A100 80 GB.
XLA_PYTHON_CLIENT_PREALLOCATE=true
XLA_CLIENT_MEM_FRACTION=0.95

%environment
export PATH="/hmmer/bin:/alphafold3_venv/bin:$PATH"
export XLA_FLAGS="--xla_gpu_enable_triton_gemm=false"
export XLA_PYTHON_CLIENT_PREALLOCATE=true
export XLA_CLIENT_MEM_FRACTION=0.95
%runscript
cd /app/alphafold
exec /bin/bash python3 run_alphafold.py "$@"
%startscript
cd /app/alphafold
exec /bin/bash python3 run_alphafold.py "$@"
```

The building process requires a CUDA module:

``` 
module load CUDA/12.5.0
apptainer build AF3.sif AF3.def 
```

This software requires model parameters subject to terms of use (described in the
repository). Once you have the installed Alphafold3 and the model parameters, you
can use a script similar to this:

``` slurm
#!/bin/bash
#SBATCH -A Project_ID
#SBATCH -J af3
#SBATCH -t 15:00:00
#SBATCH -c 14
#SBATCH --gpus-per-node=l40s:1
#SBATCH -o output_%j.out          # output file
#SBATCH -e error_%j.err           # error messages
 
# Clean the environment from loaded modules
ml purge > /dev/null 2>&1
 
module load CUDA/12.5.0
 
export ALPHAFOLD_HHBLITS_N_CPU=14
 
apptainer exec \
     --nv \
     --bind ./:/root/af_input \
     --bind ./:/root/af_output \
     --bind /folder-where-you-clonned/alphafold3/modelparameters:/root/models \
     --bind /folder-where-you-clonned/alphafold3/databases:/root/public_databases \
    /folder-where-you-clonned/alphafold3/AF3.sif  \
     python /folder-where-you-clonned/alphafold3/run_alphafold.py \
     --json_path=/root/af_input/dac_shya.json \
     --model_dir=/root/models \
     --db_dir=/root/public_databases \
     --output_dir=/root/af_output
```

to run this software. 
