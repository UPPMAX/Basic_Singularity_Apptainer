# Singularity and MPI applications

> The Singularity documentation is excellent starting point - [link](https://sylabs.io/guides/3.8/user-guide/mpi.html)     
The C3SE Singularity has really nice summary as well - [link](https://www.c3se.chalmers.se/documentation/applications/containers/)

# **Host based or Hybrid mode**

Here is an example of simple MPI program compiled with OpenMPI 4.1.2 in Ubuntu 22.04 container.


```singularity linenums="1" hl_lines="14"
Bootstrap:  docker
From: ubuntu:22.04

%setup
  mkdir -p ${SINGULARITY_ROOTFS}/opt/mpi-test

%files
  mpi-test.c /opt/mpi-test/

%post
  apt-get update && apt-get install -y build-essential mpi-default-dev lsof slurm-client \
  && rm -rf /var/lib/apt/lists/*
  
  cd /opt/mpi-test && mpicc -o mpi-test mpi-test.c

%runscript
  /bin/bash
```

??? note "mpi-test.c"
    ```c++
    #include <mpi.h>
    #include <stdio.h>

    int main(int argc, char** argv) {
        // Initialize the MPI environment
        MPI_Init(NULL, NULL);

        // Get the number of processes
        int world_size;
        MPI_Comm_size(MPI_COMM_WORLD, &world_size);

        // Get the rank of the process
        int world_rank;
        MPI_Comm_rank(MPI_COMM_WORLD, &world_rank);

        // Get the name of the processor
        char processor_name[MPI_MAX_PROCESSOR_NAME];
        int name_len;
        MPI_Get_processor_name(processor_name, &name_len);

        // Print off a hello world message
        printf("Hello world from processor %s, rank %d out of %d processors\n",
              processor_name, world_rank, world_size);

        // Finalize the MPI environment.
        MPI_Finalize();
    }
    ```

## **Running on single node** 

Running MPI on a single machine is almost trivial, since the communication between the processes is done locally. Below are two different scenarios.

> **Host based MPI runtime** (_note that the host is running the same OpenMPI version_)
```bash hl_lines="1"
HP-Z2:> mpirun -n 4 singularity exec mpi-test.sif /opt/mpi-test/mpi-test

Authorization required, but no authorization protocol specified
Authorization required, but no authorization protocol specified
Hello world from processor HP-Z2, rank 1 out of 4 processors
Hello world from processor HP-Z2, rank 3 out of 4 processors
Hello world from processor HP-Z2, rank 0 out of 4 processors
Hello world from processor HP-Z2, rank 2 out of 4 processors
```

> **Container based MPI runtime**
```bash hl_lines="1"
HP-Z2:> singularity exec mpi-test.sif mpirun -n 4 /opt/mpi-test/mpi-test

Authorization required, but no authorization protocol specified
Authorization required, but no authorization protocol specified
Hello world from processor HP-Z2, rank 3 out of 4 processors
Hello world from processor HP-Z2, rank 1 out of 4 processors
Hello world from processor HP-Z2, rank 2 out of 4 processors
Hello world from processor HP-Z2, rank 0 out of 4 processors
```  
---


## **Running singularity with MPI across multiple nodes**

Running on multiple nodes is a bit of challenge. `mpirun` needs to know about the allocated resources and your program should be compiled to support the network hardware...

> **Host based MPI runtime** (_example for running on Rackham@UPPMAX_)
```slurm hl_lines="8"
#!/bin/bash -l
#SBATCH -A project
#SBATCH -n 40 -p devel
#SBATCH -t 5:00

module load gcc/11.3.0 openmpi/4.1.2

mpirun singularity exec mpi-test.sif /opt/mpi-test/mpi-test
```
Note: `openmpi` package from the Ubuntu distribution is compiled without `SLURM` support and the executable has problems to run `srun` - [link](https://www.open-mpi.org/faq/?category=slurm)

??? note "output"
    ```
    Hello world from processor r484.uppmax.uu.se, rank 8 out of 40 processors
    Hello world from processor r484.uppmax.uu.se, rank 2 out of 40 processors
    Hello world from processor r484.uppmax.uu.se, rank 3 out of 40 processors
    Hello world from processor r484.uppmax.uu.se, rank 4 out of 40 processors
    Hello world from processor r484.uppmax.uu.se, rank 9 out of 40 processors
    Hello world from processor r484.uppmax.uu.se, rank 0 out of 40 processors
    Hello world from processor r484.uppmax.uu.se, rank 1 out of 40 processors
    Hello world from processor r484.uppmax.uu.se, rank 5 out of 40 processors
    Hello world from processor r484.uppmax.uu.se, rank 6 out of 40 processors
    Hello world from processor r484.uppmax.uu.se, rank 7 out of 40 processors
    Hello world from processor r484.uppmax.uu.se, rank 10 out of 40 processors
    Hello world from processor r484.uppmax.uu.se, rank 11 out of 40 processors
    Hello world from processor r484.uppmax.uu.se, rank 12 out of 40 processors
    Hello world from processor r484.uppmax.uu.se, rank 13 out of 40 processors
    Hello world from processor r484.uppmax.uu.se, rank 14 out of 40 processors
    Hello world from processor r484.uppmax.uu.se, rank 15 out of 40 processors
    Hello world from processor r484.uppmax.uu.se, rank 16 out of 40 processors
    Hello world from processor r484.uppmax.uu.se, rank 17 out of 40 processors
    Hello world from processor r484.uppmax.uu.se, rank 18 out of 40 processors
    Hello world from processor r484.uppmax.uu.se, rank 19 out of 40 processors
    Hello world from processor r485.uppmax.uu.se, rank 20 out of 40 processors
    Hello world from processor r485.uppmax.uu.se, rank 23 out of 40 processors
    Hello world from processor r485.uppmax.uu.se, rank 25 out of 40 processors
    Hello world from processor r485.uppmax.uu.se, rank 26 out of 40 processors
    Hello world from processor r485.uppmax.uu.se, rank 29 out of 40 processors
    Hello world from processor r485.uppmax.uu.se, rank 33 out of 40 processors
    Hello world from processor r485.uppmax.uu.se, rank 24 out of 40 processors
    Hello world from processor r485.uppmax.uu.se, rank 27 out of 40 processors
    Hello world from processor r485.uppmax.uu.se, rank 28 out of 40 processors
    Hello world from processor r485.uppmax.uu.se, rank 31 out of 40 processors
    Hello world from processor r485.uppmax.uu.se, rank 32 out of 40 processors
    Hello world from processor r485.uppmax.uu.se, rank 37 out of 40 processors
    Hello world from processor r485.uppmax.uu.se, rank 39 out of 40 processors
    Hello world from processor r485.uppmax.uu.se, rank 21 out of 40 processors
    Hello world from processor r485.uppmax.uu.se, rank 22 out of 40 processors
    Hello world from processor r485.uppmax.uu.se, rank 30 out of 40 processors
    Hello world from processor r485.uppmax.uu.se, rank 35 out of 40 processors
    Hello world from processor r485.uppmax.uu.se, rank 36 out of 40 processors
    Hello world from processor r485.uppmax.uu.se, rank 38 out of 40 processors
    Hello world from processor r485.uppmax.uu.se, rank 34 out of 40 processors
    ```


> **Container based MPI runtime**
```slurm hl_lines="8"
#!/bin/bash -l
#SBATCH -A project
#SBATCH -n 40 -p devel
#SBATCH -t 5:00

module load gcc/11.3.0 openmpi/4.1.2

singularity exec mpi-test.sif  mpirun --launch-agent 'singularity exec /FULL_PATH/mpi-test.sif orted' /opt/mpi-test/mpi-test
```

??? note "still failing"
    ```
    --------------------------------------------------------------------------
    An ORTE daemon has unexpectedly failed after launch and before
    communicating back to mpirun. This could be caused by a number
    of factors, including an inability to create a connection back
    to mpirun due to a lack of common network interfaces and/or no
    route found between them. Please check network connectivity
    (including firewalls and network routing requirements).
    --------------------------------------------------------------------------
    ```


# **Binding mode**

It is also possible to run MPI programs with Apptainer where the MPI library is only present in the
Host. An advantage of this mode is that the size of the final image can be reduced. A possible drawback is that one
will need to bind manually the required libraries for the MPI program. 

In the following paragraphs, we will show how to run a simple MPI example in the binding mode. This program
prints out the ranks IDs out of the total number of ranks requested.

??? note "hello_mpi.c"
    ```c++
    #include <mpi.h>
    #include <stdio.h>
    
    int main(int argc, char** argv) {
        MPI_Init(&argc, &argv);
        int rank, size;
        MPI_Comm_rank(MPI_COMM_WORLD, &rank);
        MPI_Comm_size(MPI_COMM_WORLD, &size);
        printf("Hello from rank %d out of %d\n", rank, size);
        MPI_Finalize();
        return 0;
    }
    ```


> **Load the OpenMPI modules and compile (Kebnekaise)**
```
ml GCC/13.3.0 OpenMPI/5.0.3

mpicc hello_mpi.c -o hello_mpi
```

The list of dynamic dependencies for the executable can be found as follows:

 
> **linked libraries**
```
ldd hello_mpi

Output (Kebnekaise):
	linux-vdso.so.1 (0x00007uauao3cb000)
	libmpi.so.40 => /software/OpenMPI/5.0.3-GCC-13.3.0/lib/libmpi.so.40 (0x00007oauaoe42e000)
	libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007aouaoec223000)
        ...
```

Write a batch script where all the paths to the dependencies are included:

```slurm hl_lines="8"
#!/bin/bash
#SBATCH -A hpc2n202u-xyz
#SBATCH -N 2
#SBATCH -n 28
#SBATCH -t 00:03:00

ml GCC/13.3.0 OpenMPI/5.0.3

export UCX_NET_DEVICES=all
export UCX_TLS=tcp
mpirun -np 28 apptainer exec --bind /hpc2n/eb/software/OpenMPI/5.0.3-GCC-13.3.0 --bind /hpc2n/eb/software/UCC/1.3.0-GCCcore-13.3.0 --bind /hpc2n/eb/software/UCX/1.16.0-GCCcore-13.3.0 --bind /hpc2n/eb/software/binutils/2.42-GCCcore-13.3.0 --bind /hpc2n/eb/software/PMIx/5.0.2-GCCcore-13.3.0 --bind /lib/x86_64-linux-gnu --bind /lib --bind /hpc2n/eb/software/libevent/2.1.12-GCCcore-13.3.0 --bind /hpc2n/eb/software/hwloc/2.10.0-GCCcore-13.3.0  mpi_test.sif /opt/app/hello_mpi
```

The UCX variables are used to detect all network interfaces. A possible output looks like this:


??? note "output"
    ```
    Warning: program compiled against libxml 212 using older 209
    Warning: program compiled against libxml 212 using older 209
    Warning: program compiled against libxml 212 using older 209
    Warning: program compiled against libxml 212 using older 209
    Warning: program compiled against libxml 212 using older 209
    Warning: program compiled against libxml 212 using older 209
    Warning: program compiled against libxml 212 using older 209
    Warning: program compiled against libxml 212 using older 209
    Warning: program compiled against libxml 212 using older 209
    Warning: program compiled against libxml 212 using older 209
    Warning: program compiled against libxml 212 using older 209
    Warning: program compiled against libxml 212 using older 209
    Warning: program compiled against libxml 212 using older 209
    Warning: program compiled against libxml 212 using older 209
    Warning: program compiled against libxml 212 using older 209
    Warning: program compiled against libxml 212 using older 209
    Warning: program compiled against libxml 212 using older 209
    Warning: program compiled against libxml 212 using older 209
    Warning: program compiled against libxml 212 using older 209
    Warning: program compiled against libxml 212 using older 209
    Warning: program compiled against libxml 212 using older 209
    Warning: program compiled against libxml 212 using older 209
    Warning: program compiled against libxml 212 using older 209
    Warning: program compiled against libxml 212 using older 209
    Warning: program compiled against libxml 212 using older 209
    Warning: program compiled against libxml 212 using older 209
    Warning: program compiled against libxml 212 using older 209
    Warning: program compiled against libxml 212 using older 209
    Hello from rank 11 out of 28
    Hello from rank 0 out of 28
    Hello from rank 10 out of 28
    Hello from rank 13 out of 28
    Hello from rank 12 out of 28
    Hello from rank 15 out of 28
    Hello from rank 17 out of 28
    Hello from rank 19 out of 28
    Hello from rank 2 out of 28
    Hello from rank 9 out of 28
    Hello from rank 18 out of 28
    Hello from rank 7 out of 28
    Hello from rank 14 out of 28
    Hello from rank 16 out of 28
    Hello from rank 5 out of 28
    Hello from rank 3 out of 28
    Hello from rank 6 out of 28
    Hello from rank 8 out of 28
    Hello from rank 1 out of 28
    Hello from rank 22 out of 28
    Hello from rank 4 out of 28
    Hello from rank 20 out of 28
    Hello from rank 21 out of 28
    Hello from rank 23 out of 28
    Hello from rank 24 out of 28
    Hello from rank 27 out of 28
    Hello from rank 25 out of 28
    Hello from rank 26 out of 28
    ```
