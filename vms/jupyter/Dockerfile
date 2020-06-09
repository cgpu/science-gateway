# Heavily borrowed from docker-stacks/scipy-notebook/
# https://github.com/jupyter/docker-stacks/blob/master/scipy-notebook/Dockerfile

ARG BASE_CONTAINER=jupyter/datascience-notebook@sha256:e690099c57a65a8efdd59534f22f5b3fe4bd7a8139bf095142f46cf52079846a
FROM $BASE_CONTAINER

LABEL maintainer="Unidata <support-gateway@unidata.ucar.edu>"

SHELL ["/bin/bash", "-o", "pipefail", "-c"]

USER root

RUN apt-get update && \
    apt-get install -y --no-install-recommends zlib1g-dev libssl-dev && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

USER $NB_UID

RUN conda install --quiet --yes \
    'conda-forge::nb_conda_kernels'  && \
    conda clean --all -f -y && \
    jupyter lab clean -y && \
    npm cache clean --force

COPY README_FIRST.ipynb /

COPY SAIGE.yml /
RUN conda env create --name SAIGE --file /SAIGE.yml && \
    source /opt/conda/etc/profile.d/conda.sh  && \
    conda activate SAIGE && \
    pip install 'cget==0.1.9' 'click==7.1.2' && \
    Rscript -e 'Sys.setenv(TAR = "/bin/tar"); devtools::install_github("weizhouUMICH/SAIGE@8f3021d")' && \
    conda clean --all -y

COPY append_jupyter_notebook_config_py.txt /
RUN cat /append_jupyter_notebook_config_py.txt >> /home/jovyan/.jupyter/jupyter_notebook_config.py

USER $NB_UID
