# Running Singularity containers from online repositories

Let us try to run a small and simple container from Docker Hub repository. Singularity, will pull the docker image in the cache, convert it and run it. The output should look something like this:

!!! warning "Important - reminder"
    Environmental variables that will help you to redirect potentially large folders to alternative location - keep in mind that your `$HOME` folder is relatively small in size.

    ```bash
    export SINGULARITY_CACHEDIR=/proj/nais-XXX-XX/nobackup/SINGULARITY_CACHEDIR
    export APPTAINER_CACHEDIR=${SINGULARITY_CACHEDIR}

    mkdir -p $APPTAINER_CACHEDIR
    ```


```
singularity run docker://sylabsio/lolcow

INFO:    Converting OCI blobs to SIF format
INFO:    Starting build...
INFO:    Fetching OCI image...
27.2MiB / 27.2MiB [====================================================================================================] 100 % 8.5 MiB/s 0s
45.8MiB / 45.8MiB [====================================================================================================] 100 % 8.5 MiB/s 0s
INFO:    Extracting OCI image...
INFO:    Inserting Singularity configuration...
INFO:    Creating SIF file...
 ________________________________
< Thu Oct 2 09:35:31 Europe 2025 >
 --------------------------------
        \   ^__^
         \  (oo)\_______
            (__)\       )\/\
                ||----w |
                ||     ||
```
The container executes predefined command `date | cowsay | lolcat`.

- `date` - print current date and time.
- `cowsay` - configurable speaking/thinking cow (and a bit more)
- `lolcat` - rainbow coloring effect for text console display

Let's run it again.
```
singularity run docker://sylabsio/lolcow

INFO:    Using cached SIF image
 ________________________________
< Thu Oct 2 09:37:31 Europe 2025 >
 --------------------------------
        \   ^__^
         \  (oo)\_______
            (__)\       )\/\
                ||----w |
                ||     ||
```
Note, that singularity, after contacting the repositories, realizes that the container is in the local cache and proceeds to run it. But where is it?

[More details...](https://sylabs.io/guides/latest/user-guide/singularity_and_docker.html)

``` 
singularity cache list

There are 1 container file(s) using 87.96 MiB and 8 oci blob file(s) using 99.09 MiB of space
Total space used: 187.04 MiB
```
!!! info
    Over time the cache will grow and might easily accumulate unnecessary "blobs". To clean the cache you can run.
    ``` bash
    singularity cache clean
    ```
    Here is how the cache might look like:
    ``` bash
    singularity cache clean --dry-run
    User requested a dry run. Not actually deleting any data!
    INFO:    Removing blob cache entry: blobs
    INFO:    Removing blob cache entry: index.json
    INFO:    Removing blob cache entry: oci-layout
    INFO:    No cached files to remove at /home/ubuntu/.singularity/    cache/library
    INFO:    Removing oci-tmp cache entry:     a692b57abc43035b197b10390ea2c12855d21649f2ea2cc28094d18b93360eeb
    INFO:    No cached files to remove at /home/ubuntu/.singularity/    cache/shub
    INFO:    No cached files to remove at /home/ubuntu/.singularity/    cache/oras
    INFO:    No cached files to remove at /home/ubuntu/.singularity/    cache/net
    ```


## More examples

```
singularity run docker://dctrud/wttr
```
![output](images/wttr.png)


!!! INFO
    ![Coderefinery](./images/coderefinery.png){ width="40", align=left } Excellent course material by the [CodeRefinery](https://coderefinery.org/) project  
    [Containers on HPC with Apptainer](https://coderefinery.github.io/hpc-containers/)

### metaWRAP - a flexible pipeline for genome-resolved metagenomic data analysis

Here is an example how to use the metaWRAP pipeline from the docker container - [installation instructions](https://github.com/bxlab/metaWRAP#docker-installation).

``` bash
# Original instructions (do NOT run)
docker pull quay.io/biocontainers/metawrap:1.2--1
```
In this particular case it is as easy as:

```
singularity pull docker://quay.io/biocontainers/metawrap:1.2--1

INFO:    Converting OCI blobs to SIF format
INFO:    Starting build...
Getting image source signatures
...
```
This will bring the docker image locally and covert it to Singularity format.

Then, one can start the container and use it interactively.  
In this particular case, executing the Singularity container gives us a shell running in the container.

```
./metawrap_1.2--1.sif
WARNING: Skipping mount /usr/local/var/singularity/mnt/session/etc/resolv.conf [files]: /etc/resolv.conf doesn't exist in container

Singularity> metawrap --version
metaWRAP v=1.2
```

To run the tool from the command line (outside of the container, as you would use it in scripts) we need to add the call for the tool.

Original commad in the cript:  
```$ metawrap binning -o Lanna-straw_initial_binning_concoct -t 20 -a /proj/test/megahit_ass_Lanna-straw/final.contigs.fa --concoct --run-checkm /proj/test/Lanna-straw_reads_trimmed/*.fastq```

The command now calls the tool from the Singularity container:  
```singularity exec metawrap_1.2--1.sif metawrap binning -o Lanna-straw_initial_binning_concoct -t 20 -a /proj/test/megahit_ass_Lanna-straw/final.contigs.fa --concoct --run-checkm /proj/test/Lanna-straw_reads_trimmed/*.fastq```

!!! info "Pulling Singularity container from online or local library/repository"
    - **library://** to build from the [Container Library](https://    cloud.sylabs.io/library)  
    ` library://sylabs-jms/testing/lolcow`
    - **docker://** to build from [Docker Hub](https://hub.docker.com/    )  
    ` docker://godlovedc/lolcow`
    - **shub://** to build from [Singularity Hub](https://    singularityhub.com/)
    - path to a existing container on your local machine
    - path to a directory to build from a sandbox
    - path to a Singularity definition file

### Tensorflow 
Let's have some tensorflow running. First `pull` the image from docker hub (~2.6GB).  

```
singularity pull docker://tensorflow/tensorflow:latest-gpu

INFO:    Converting OCI blobs to SIF format
INFO:    Starting build...
Getting image source signatures
...
```

If you have a GPU card, here is how easy you can get tensorflow running. Note the `--nv` option on the command line.

---

``` python
singularity exec --nv tensorflow_latest-gpu.sif python3

Python 3.8.10 (default, Nov 26 2021, 20:14:08) 
[GCC 9.3.0] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> import tensorflow as tf
>>> tf.config.list_physical_devices('GPU')
[PhysicalDevice(name='/physical_device:GPU:0', device_type='GPU')]
```

## [NVIDIA Deep Learning Frameworks](./PyTorch_NVIDIA.md) - additional material.
