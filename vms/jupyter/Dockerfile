# Heavily borrowed from docker-stacks/scipy-notebook/
# https://github.com/jupyter/docker-stacks/blob/master/scipy-notebook/Dockerfile

ARG BASE_CONTAINER=jupyter/scipy-notebook:latest
FROM $BASE_CONTAINER

LABEL maintainer="Unidata <support-gateway@unidata.ucar.edu>"

SHELL ["/bin/bash", "-o", "pipefail", "-c"]

USER root

RUN apt-get update && \
    apt-get install -y --no-install-recommends ffmpeg vim curl && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

USER $NB_UID

RUN conda install --quiet --yes \
    'conda-forge::nb_conda_kernels'  && \
    wget https://raw.githubusercontent.com/Unidata/python-training/master/environment.yml && \
    conda env update --name python-training -f environment.yml && \
    rm environment.yml && \
    pip install --no-cache-dir nbgitpuller && \
    conda clean --all -f -y && \
    jupyter lab clean -y && \
    npm cache clean --force && \
    rm -rf /home/$NB_USER/.cache/yarn && \
    rm -rf /home/$NB_USER/.node-gyp && \
    fix-permissions $CONDA_DIR && \
    fix-permissions /home/$NB_USER

COPY README_FIRST.ipynb /
COPY .condarc /

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
