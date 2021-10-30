# VSCode Development Container Template

This template enables you to use a full-fledged containerized development environment for your machine learning projects - all with VSCode!

The container itself will only take care of running the code, the files/data and your credentials are brought over from and saved to your local workspace, so it all works out of the box.

## Prerequisites

You should have the following installed on the machine where your containers will run, as well as on your local machine (if they are different):

- [docker](https://docs.docker.com/get-docker/)
- [docker-compose](https://docs.docker.com/compose/install/)
- [VSCode](https://code.visualstudio.com/docs/setup/setup-overview)

If you plan to use GPU in the containers you will also need:

- [NVIDIA Docker container runtime](https://github.com/NVIDIA/nvidia-docker)
- [NVIDIA drivers](https://github.com/NVIDIA/nvidia-docker/wiki/Frequently-Asked-Questions#how-do-i-install-the-nvidia-driver)

If you don't plan to use the GPU, then you need to make a few small adjustments described in the [CPU Only](#cpu-only) section.

Additionally, you need to expose your *user id* as an environmental variable `UID` on the device where your containers will run. If you use bash or zsh, you do this by adding the following to your `.bashrc` or `.zshrc` file:

```
export UID
```

## Quickstart: local

This assumes that all your files are on a local machine, where you also want to run the container. If you want to run the container on a remote machine, check out the [remote](#quickstart-remote) section.

All you need to do is to open the repository in VSCode, press <kbd>Ctrl</kbd> + <kbd>Shift</kbd> + <kbd>P</kbd> and type/select "Reopen in Container".

## Quickstart: remote

This instructions are for the following scenario: your files and credentials are on a remote **host** machine (such as an AWS server, desktop workstation), and the only use of your **local** machine is to connect to the host. It is required that you have **the same username** on both local and host machine.

> The same username is required, because it will be used in the name of docker compose project, as well as the docker image. This enables multiple users on the same machine to use development containers without interfering with each other, as they will all have a separate compose project/docker image.

First, you need to set the `docker.host` setting in VSCode on your local machine to point at your host machine - see [here](https://code.visualstudio.com/docs/remote/containers-advanced#_a-basic-remote-example) for instructions. Next, you need to execute the following on both the local and host machine from the root of your repository (this you have to do just once in the lifecycle of the project)

```
echo "COMPOSE_PROJECT_NAME=${PWD##*/}_${USER}" >> .env 
```

Next, execute this from the root of your repository on the host machine

``` 
docker-compose -f .devcontainer/docker-compose.yml up -d
```

You can close the remote terminal after that. Finally, open the repository in VSCode on your local machine, press <kbd>Ctrl</kbd> + <kbd>Shift</kbd> + <kbd>P</kbd> and type/select "Reopen in Container".

This way, everything will work as expected - even the port 8888 of the remote container will be mapped to port 8888 in your local machine.

## What's in this template

This template does a few things, and it's useful to know what they are, so you know what to adjust. I'll explain it by describing what is the function of each file in this repo. Here's a directory tree diagram:

``` 
.
├── .devcontainer
│   ├── devcontainer.json
│   ├── docker-compose.yml
│   ├── Dockerfile
│   ├── env_dev.yml
│   └── jupyter_lab_config.py
├── env.yml
├── sys_requirements.txt
└── ...
```

- `.devcontainer/devcontainer.json`: This defines the VSCode development container. It delegates the "base" of the container itself to `docker-compose.yml` , and focuses on forwarding ports, installing VSCode extensions and adjusting VSCode settings.
- `.devcontainer/docker-compose.yml`: This file mainly takes care of configuring how the docker container connects to the local file system. Namely, it does these things:
  - sets the user ID of the user in the container to match the local user, to avoid file permission issues
  - mounts the workspace folder to the container
  - mounts the credentials folders to the container (as read-only): for example, the `.ssh` and `.aws` folders
  - mounts some other folders (in this example the DVC cache) and sets environmental variables credentials
- `.devcontainer/env_dev.yml`: This conda environment file specifies the requirements for development (they will be added to the base environment) - for example testing and linting.
- `.devcontainer/Dockerfile`: This is the real "meat" of this whole thing. It creates a container based on the base CUDA image (which by itself does not have drivers or CUDA installed), installs all the system and python requirements and creates a user corresponding to your current local user. If your setup requires some heavier system modification, you should do it here.
- `.devcontainer/jupyter_lab_config.py`: this sets some useful jupyter notebook/lab presets, such as a password (you should change this) and the default port.
- `env.yml`: This is a conda environment, defining all the base (non-development) requirements of your project.
- `sys_requirements.txt`: This is a minimal system requirements (stuff you install with `apt-get` ) file.

## CPU Only

If you want to use this on a CPU-only device, you need to make three minor changes:

1. In `env.yml` change `cudatoolkit=11.0` to `cpuonly`.
2. In `.devcontainer/docker-compose.yml` remove `runtime: nvidia`
3. In `Dockerfile`, change the `nvidia/cuda...` base image to `ubuntu:${UBUNTU_VERSION}`
