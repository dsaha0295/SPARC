# Base image with R and tidyverse
FROM rocker/tidyverse:latest

# Avoid interactive prompts during package installation
ENV DEBIAN_FRONTEND=noninteractive

# System-level dependencies
RUN apt-get update && apt-get install -y \
    libhdf5-dev \
    libcurl4-openssl-dev \
    libssl-dev \
    libxml2-dev \
    libgit2-dev \
    libglpk-dev \
    libfontconfig1-dev \
    libfreetype6-dev \
    libpng-dev \
    libtiff5-dev \
    libjpeg-dev \
    libharfbuzz-dev \
    libfribidi-dev \
    libzstd-dev \
    python3 \
    python3-pip \
    && apt-get clean && rm -rf /var/lib/apt/lists/*

# Install core R packages
RUN R -e "install.packages(c('remotes', 'data.table', 'patchwork', 'ggpubr', 'devtools', 'future', 'reshape2', 'rlist', 'hdf5r'))"

# Install Seurat and SeuratDisk
RUN R -e "install.packages('Seurat')"
RUN R -e "remotes::install_github('mojaveazure/seurat-disk')"

# Install CopyKAT
RUN R -e "remotes::install_github('navinlabcode/copykat')"

# Install Bioconductor packages for inferCNV
RUN R -e "if (!requireNamespace('BiocManager', quietly=TRUE)) install.packages('BiocManager')"
RUN R -e "BiocManager::install(c('SummarizedExperiment', 'complexheatmap', 'HDF5Array', 'GenomicRanges', 'BiocGenerics'))"

# Install inferCNV from GitHub
RUN R -e "BiocManager::install('infercnv')"

# Install Biomart
RUN R -e "BiocManager::install(c('biomaRt'))"


# Install Biomart
RUN R -e "BiocManager::install(c('ggsci'))"




