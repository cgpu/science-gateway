# Heavily borrowed from docker-stacks/scipy-notebook/
# https://github.com/jupyter/docker-stacks/blob/master/scipy-notebook/Dockerfile

ARG BASE_CONTAINER=jupyter/datascience-notebook@sha256:e690099c57a65a8efdd59534f22f5b3fe4bd7a8139bf095142f46cf52079846a
FROM $BASE_CONTAINER

LABEL maintainer="Unidata <support-gateway@unidata.ucar.edu>"

SHELL ["/bin/bash", "-o", "pipefail", "-c"]

USER root

COPY .condarc /home/jovyan/

RUN apt-get update && \
    apt-get install -y --no-install-recommends zlib1g-dev libssl-dev && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

USER $NB_UID

RUN conda install --quiet --yes \
    'conda-forge::nb_conda_kernels'  \
    jupytext && \
    conda clean --all -f -y && \
    jupyter lab clean -y && \
    npm cache clean --force

COPY README_FIRST.ipynb /

COPY SAIGE.yml /
RUN conda env create --name SAIGE --file /SAIGE.yml && \
    source /opt/conda/etc/profile.d/conda.sh  && \
    conda activate SAIGE && \
    pip install 'cget==0.1.9' 'click==7.1.2' && \
    Rscript -e 'Sys.setenv(TAR = "/bin/tar"); devtools::install_github("weizhouUMICH/SAIGE@8f3021d", build_manual = TRUE )' && \
    conda clean --all -y

COPY extras_jupy_nbook_config.py /
RUN cat /extras_jupy_nbook_config.py >> /home/jovyan/.jupyter/jupyter_notebook_config.py

USER root

# Found here: https://stackoverflow.com/questions/56619347/anaconda-environment-bash-prefix-too-long
RUN conda config --set env_prompt '($(basename {default_env})) '

USER $NB_UID


