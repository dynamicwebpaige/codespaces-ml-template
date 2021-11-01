FROM jupyter/datascience-notebook

# We want to run common-debian.sh from here:
# https://github.com/microsoft/vscode-dev-containers/tree/main/script-library#development-container-scripts
# But that script assumes that the main non-root user (in this case jovyan)
# is in a group with the same name (in this case jovyan).  So we must first make that so.
COPY library-scripts/common-debian.sh /tmp/library-scripts/
USER root
RUN apt-get update \
 && groupadd jovyan \
 && usermod -a -G jovyan jovyan \
 && bash /tmp/library-scripts/common-debian.sh \
 && apt-get clean -y && rm -rf /var/lib/apt/lists/* /tmp/library-scripts

RUN pip install conda

# ARG UBUNTU_VERSION=20.04
# ARG CUDA_VERSION=11.0
# FROM continuumio/miniconda3 AS conda

COPY env.yml /
COPY .devcontainer/env_dev.yml /
RUN conda update conda -c conda-forge && \
    conda env update -f /env.yml -n base && \
    conda env update -f /env_dev.yml -n base && \
    conda clean -afy

# FROM ubuntu${UBUNTU_VERSION}

# Prepare shell and file system
ENV LANG=C.UTF-8 LC_ALL=C.UTF-8 SHELL=/bin/bash
ENV PATH /opt/conda/bin:$PATH
SHELL ["/bin/bash", "-c"]

# Install all system stuff, including node
COPY sys_requirements.txt /tmp
ARG DEBIAN_FRONTEND="noninteractive"
RUN apt update && apt install -y --no-install-recommends \
    $(cat tmp/sys_requirements.txt) && \
    apt clean && rm -rf /var/lib/apt/lists/*

# Create the sudo user
ARG USERNAME
ARG UID
RUN useradd $USERNAME -u $UID -G sudo -s /bin/bash -m && \
    echo $USERNAME' ALL=(ALL) NOPASSWD:ALL' >> /etc/sudoers
USER $USERNAME
ENV HOME=/home/$USERNAME
RUN sudo mkdir /ws && sudo chown -R $USERNAME /ws

# Copy over conda and bashrc, install environment
COPY --from=conda --chown=$USERNAME /opt/ /opt/
RUN sudo ln -s /opt/conda/etc/profile.d/conda.sh /etc/profile.d/conda.sh && \
    echo ". /opt/conda/etc/profile.d/conda.sh" >> ~/.bashrc && \
    echo "conda activate base" >> ~/.bashrc

# Jupyter config
COPY --chown=$USERNAME .devcontainer/jupyter_lab_config.py $HOME/.jupyter/ 

# Prepare entrypoint and mount folder
WORKDIR /ws
