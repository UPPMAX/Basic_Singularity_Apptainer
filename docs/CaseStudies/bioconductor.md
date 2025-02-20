# Bulk RNA-seq analysis
Credits to [Andrey](https://github.com/flerpan01)

Singularity image with pre-installed libraries for running bulk RNA-seq analysis


```singularity
Bootstrap: docker
From: bioconductor/bioconductor_docker:latest

%labels
Author https://github.com/flerpan01

%environment
export LC_ALL=C

%post
# CRAN packages
cat << EOF > pkgs-CRAN.txt
ggplot2
gtools
tools
scales
data.table
ggrepel
forcats
RColorBrewer
cowplot
ggsignif
openxlsx
patchwork
EOF

R -e "install.packages(readLines('pkgs-CRAN.txt'))"

# Bioconductor packages
cat << EOF > pkgs-Bioc.txt
clusterProfiler
edgeR
DESeq2
tximport
biomaRt
ComplexHeatmap
org.Mm.eg.db
EOF

R -e "BiocManager::install(readLines('pkgs-Bioc.txt'))"

%runscript
Rscript "$@"
```