# Quick start


## HPC setup
---
The setup provided by computer centers are enough for the most common tasks - running, pulling and building[^1].  

[C3SC](https://www.c3se.chalmers.se/documentation/miscellaneous/containers/),
[PDC](https://support.pdc.kth.se/doc/applications/apptainer/)*[^1],
[HPC2N](https://docs.hpc2n.umu.se/software/containers/),
[NSC](https://www.nsc.liu.se/support/singularity/),
[Lunarc](https://docs.hpc2n.umu.se/software/containers/).

!!! warning "Important"
    > Please setup the following environmental variables that will help you to redirect potentially large files to alternative locations.
    Keep in mind that your `$HOME` folder is relatively small in size. Consider adding the settings in your `~/.bashrc` file.

    ```bash
    export SINGULARITY_CACHEDIR=/proj/nais-XXXX-XX/nobackup/SINGULARITY_CACHEDIR
    ```
    ```bash
    export APPTAINER_CACHEDIR=${SINGULARITY_CACHEDIR}

    mkdir -p $APPTAINER_CACHEDIR
    ```


## Local setup
---
This is the preferred setup if you plan to experiment and build containers yourself.


### Apptainer
- [Install from pre-built packages](https://apptainer.org/docs/admin/1.3/installation.html#install-from-pre-built-packages)
- [Installation on Windows or Mac](https://apptainer.org/docs/admin/1.3/installation.html#installation-on-windows-or-mac)

!!! note
    There is a possibility to [install Apptainer as unprivileged user](https://apptainer.org/docs/admin/1.3/installation.html#install-unprivileged-from-pre-built-binaries), providing that the requirements are satisfied.

### Singularity 
- Linux pre-built packages are provided for `.deb` and `.rpm`based distributions [link](https://github.com/sylabs/singularity/releases)
  ``` bash
  # Ubuntu 24.04
  wget https://github.com/sylabs/singularity/releases/download/v4.3.3/singularity-ce_4.3.3-noble_amd64.deb
  sudo apt install ./singularity-ce_4.3.3-noble_amd64.deb

  # RHEL/CentOS/AlmaLinux/Rocky 9
  wget https://github.com/sylabs/singularity/releases/download/v4.3.3/singularity-ce-4.3.3-1.el9.x86_64.rpm
  sudo yum install ./singularity-ce-4.3.3-1.el9.x86_64.rpm
  ```

### Installation on Windows or Mac
[https://sylabs.io/guides/latest/admin-guide/installation.html#installation-on-windows-or-mac](https://sylabs.io/guides/latest/admin-guide/installation.html#installation-on-windows-or-mac){target=_blank}

> (PM) I have successfully installed Singularity under [WSL2](https://docs.microsoft.com/en-us/windows/wsl/install-win10){target=_blank}, but can't guarantee that it will work in all cases.  
> Look at this [page](./vagrant_windows.md) for tips on the typical Windows installation (MacOS is rather similar).

---



## Testing the installation

[https://sylabs.io/guides/latest/admin-guide/installation.html#testing-checking-the-build-configuration](https://sylabs.io/guides/latest/admin-guide/installation.html#testing-checking-the-build-configuration){target=_blank}

```
$ singularity 
```

This will show something similar as below

```
Usage:
  singularity [global options...] <command>

Available Commands:
  build       Build a Singularity image
  cache       Manage the local cache
  capability  Manage Linux capabilities for users and groups
  config      Manage various singularity configuration (root user only)
  delete      Deletes requested image from the library
  exec        Run a command within a container
  inspect     Show metadata for an image
  instance    Manage containers running as services
  key         Manage OpenPGP keys
  oci         Manage OCI containers
  plugin      Manage Singularity plugins
  pull        Pull an image from a URI
  push        Upload image to the provided URI
  remote      Manage singularity remote endpoints, keyservers and OCI/Docker registry credentials
  run         Run the user-defined default command within a container
  run-help    Show the user-defined help for an image
  search      Search a Container Library for images
  shell       Run a shell within a container
  sif         siftool is a program for Singularity Image Format (SIF) file manipulation
  sign        Attach digital signature(s) to an image
  test        Run the user-defined tests within a container
  verify      Verify cryptographic signatures attached to an image
  version     Show the version for Singularity

Run 'singularity --help' for more detailed usage information.
```

## Checking the configuration

```
$ singularity buildcfg
```
This will show something similar as below

```
PACKAGE_NAME=singularity
PACKAGE_VERSION=3.10.2
BUILDDIR=/root/singularity/builddir
PREFIX=/usr/local
EXECPREFIX=/usr/local
BINDIR=/usr/local/bin
SBINDIR=/usr/local/sbin
LIBEXECDIR=/usr/local/libexec
DATAROOTDIR=/usr/local/share
DATADIR=/usr/local/share
SYSCONFDIR=/usr/local/etc
SHAREDSTATEDIR=/usr/local/com
LOCALSTATEDIR=/usr/local/var
RUNSTATEDIR=/usr/local/var/run
INCLUDEDIR=/usr/local/include
DOCDIR=/usr/local/share/doc/singularity
INFODIR=/usr/local/share/info
LIBDIR=/usr/local/lib
LOCALEDIR=/usr/local/share/locale
MANDIR=/usr/local/share/man
SINGULARITY_CONFDIR=/usr/local/etc/singularity
SESSIONDIR=/usr/local/var/singularity/mnt/session
PLUGIN_ROOTDIR=/usr/local/libexec/singularity/plugin
SINGULARITY_CONF_FILE=/usr/local/etc/singularity/singularity.conf
SINGULARITY_SUID_INSTALL=1
```



[^1]: The information provided by PDC is targeted for HPC use and it does not reflect the common practices that will be focus on this workshop. 
