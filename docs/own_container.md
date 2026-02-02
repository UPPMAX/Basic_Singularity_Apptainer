# Building simple container

Let's start with a simple example.  

Here is a definition file to install the [IGV desktop](https://igv.org/doc/desktop/) program in virtual environment conveniently provided by the Ubuntu distribution via the docker hub repository.

!!! note "igv.def"
    ``` singularity
    Bootstrap: docker
    From: ubuntu:24.04

    %post
      export DEBIAN_FRONTEND=noninteractive
      
      apt-get update && apt-get -y dist-upgrade && \
      apt-get install -y x11-apps igv && \
      apt-get clean

      # Patch for old kernels like the one on Rackham running CentOS 7 - 2024.02.23
      # strip --remove-section=.note.ABI-tag /usr/lib/x86_64-linux-gnu/libQt5Core.so.5

    %runscript
      igv "$@"
    ```

``` bash
sudo singularity build igv.sif igv.def
```

Instead of igv, modify the definition file to install and run your, not necessarily graphical, program. Few tips: `gnuplot`, `grace`, `blender`, `povray`, `rasmol`  ...

