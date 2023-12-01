# Multi user conda installation

## What is it for?
We wanted to have a [conda](https://docs.conda.io/projects/miniconda/en/latest/) installation available to all users on our plateform, with the ability to create environments available to every user but also for each user to create their own private environments.

## Install
We chose the [miniforge](https://github.com/conda-forge/miniforge) distribution which provides [mamba](https://mamba.readthedocs.io/en/latest/).

### Run the miniforge installer
```
wget https://github.com/conda-forge/miniforge/releases/latest/download/Miniforge3-Linux-x86_64.sh
sudo bash Miniforge3-Linux-x86_64.sh
```
1. During the installation process, you will first be prompted with a license agreeement.
2. After agreeing, the installer then proposes to install in a default location. As you are running as root, the proposed location is likely something like `/root/miniforge3`. We chose to install in `/opt/miniforge3`.
3. Then, refuse to initialize conda (useless as you are running as root).

## Configure

### Protect and grant user-access to the installation
As per [this post on the conda discourse group](https://conda.discourse.group/t/multi-user-mamba-installation/228), "ensure that the group for the installation is (recursively) set to a group that only the administrator is a member of – or, at least, that the normal users are not members of. This is crucial because many conda packages contain files with group-writable permissions, and if your users are members of the installation’s group, they can, accidentally or otherwise, modify files they should not." To do so:
```
sudo groupadd conda
sudo chgrp -R conda /opt/miniforge3
sudo chmod 755 -R /opt/miniforge3
```

### Export CONDA_PKGS_DIR
To have users create their own environment, we need to have all users export environment variable `CONDA_PKGS_DIR` with a value like `$HOME/.conda/pkgs`. We take advantage of the default profile `/etc/profile`:
create a file as root in `/etc/profile.d/` (for example `/etc/profile.d/conda_setup.sh`), containing the following:
```
export CONDA_PKGS_DIR=$HOME/.conda/pkgs
```
Logout and back in to have it take effect (can check with `echo ${CONDA_PKGS_DIR}`).

### Initialize conda

As we do not want to have the base environment loaded by default, that is all there is to do. Conda will need be loaded each time we want to use it.

## Use conda

To use conda:
`source /opt/miniforge3/bin/activate`

The regular user can see both system-wide environments and private environments:
`conda env list`
```
# conda environments:
#
test                     /home/username/.conda/envs/test
base                  *  /opt/miniforge3
py37                     /opt/miniforge3/envs/py37

```


## Other helpful resources
- https://conda.discourse.group/t/multi-user-mamba-installation/228
- https://askubuntu.com/questions/1457726/how-and-where-to-install-conda-to-be-accessible-to-all-users
- https://github.com/matbinder/secure-multi-user-conda
