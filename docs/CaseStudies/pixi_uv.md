# [pixi](https://pixi.prefix.dev/latest/) and [uv](https://docs.astral.sh/uv/) in a container

Here is an example basic template to setup both tools in a Singularity container.  
(or this project [Pixitainer](https://github.com/RaphaelRibes/pixitainer) for an alternative approach)

```singularity
Bootstrap: docker          
From: ubuntu:24.04         
                           
%environment               
  export LC_ALL=C.utf8     
  export PYTHONNOUSERSITE=True         
                           
  export PIXI_HOME=/opt/pixi
  export PATH=/opt/pixi/bin:/opt/uv:/opt/uv/tool_bin:${PATH}
                           
%post                      
  export LC_ALL=C.utf8     
  export PYTHONNOUSERSITE=True
  export DEBIAN_FRONTEND=noninteractive
                           
  # apt ============================================
  apt update && apt-get -y install curl
                           
  # pixi ===========================================
  PIXI_PREFIX="/opt/pixi"  
  export PIXI_NO_PATH_UPDATE=1
  export PIXI_CACHE_DIR="/tmp/pixi_cache"
  export PIXI_HOME="${PIXI_PREFIX}"
  mkdir -p ${PIXI_HOME}    
                           
  curl -fsSL https://pixi.sh/install.sh | bash
  export PATH="${PIXI_PREFIX}/bin:${PATH}"
                           
  pixi global install bat  
                           
  # uv ==============================================
  UV_PREFIX="/opt/uv"      
  export UV_INSTALL_DIR="${UV_PREFIX}"
  mkdir -p ${UV_INSTALL_DIR}
  export INSTALLER_NO_MODIFY_PATH=1
                           
  curl -LsSf https://astral.sh/uv/install.sh | sh
  export PATH="${UV_PREFIX}:${UV_PREFIX}/tool_bin:${PATH}"
                           
  export UV_CACHE_DIR="/tmp/uv_cache"
  export UV_PYTHON_BIN_DIR="${UV_PREFIX}"
  export UV_PYTHON_INSTALL_DIR="${UV_PREFIX}/python"
  export UV_TOOL_DIR="${UV_PREFIX}/tool"
  export UV_TOOL_BIN_DIR="${UV_PREFIX}/tool_bin"
                           
                           
  #export UV_LINK_MODE=copy                                                                                                                           
  uv tool install qrcode   
                           
%runscript                 
#!/bin/sh                  
  if command -v $SINGULARITY_NAME > /dev/null 2> /dev/null; then                
    exec $SINGULARITY_NAME "$@"        
  else                     
    echo "# ERROR !!! Command $SINGULARITY_NAME not found in the container"     
  fi
```
