# Trusted Postgres Architect (TPA) Workshop
© Copyright EnterpriseDB UK Limited 2015-2024 - All rights reserved.

**What is Trusted Postgres Architect (TPA)?**
TPA is an orchestration tool developed by EnterpriseDB (EDB) that uses Ansible to deploy Postgres clusters according to EDB's recommendations.

TPA embodies the best practices followed by EDB, informed by many years of hard-earned experience with deploying and supporting Postgres. These recommendations are as applicable to quick testbed setups as to production environments.

Please check out the full [documentation](https://www.enterprisedb.com/docs/tpa/latest/) to learn how to use TPA.

# Quickstart TPA

## Install Depencdencies
First, you must install the various dependencies: Python 3, Python venv, git, openvpn, and patch. Installing from EDB repositories installs these for you along with the TPA packages.

Before you install TPA, you must install the required packages:

**Debian/Ubuntu**
```
sudo apt-get install python3 python3-pip python3-venv git openvpn patch
```
**Redhat, Rocky or AlmaLinux (RHEL7)**
```
sudo yum install python3 python3-pip epel-release git openvpn patch
```
**Redhat, Rocky or AlmaLinux (RHEL8)**
```
sudo yum install python36 python3-pip epel-release git openvpn patch
```

## Install TPA
After the prerequisites are installed, you can clone the repository:

```
git clone https://github.com/enterprisedb/tpa.git ~/tpa
```

Cloning creates a tpa directory in your home directory.

If you prefer to check out with SSH, use:

```
git clone ssh://git@github.com/EnterpriseDB/tpa.git ~/tpa
```
Add the bin directory to your path. You can find the bin directory in your newly created clone.

Add this line to your .bashrc file (or other profile file for your preferred shell):

```export PATH=$PATH:$HOME/tpa/bin```

You can now create a working TPA environment by running:

```tpaexec setup```

This command creates the Python virtual environment that TPA will use in future. All needed packages are installed in this environment. To test whether this was configured correctly, run:

```tpaexec selftest```

tpaexec is now installed.

## Using Docker

**Install Docker**
This tutorial deploys the example deployment onto Docker. If you don't already have Docker installed, you need to set it up.

To install Docker on Debian or Ubuntu:

```
sudo apt update
sudo apt install docker.io
```
Add your user to the docker group:

```
sudo usermod -aG docker <yourusername>
newgrp docker
```

**CgroupVersion**

Support for CgroupVersion 2 is not fully baked yet for docker sdk in ansible and related tooling. So while we recommend using a recent version of docker; we rely on CgroupVersion 1 until version 2 is fully supported. Instructions below suggest the changes to switch to CgroupVersion 1 if your platform uses CgroupVersion 2 by default.

On Linux:
```
$ echo 'GRUB_CMDLINE_LINUX=systemd.unified_cgroup_hierarchy=false' > \
  /etc/default/grub.d/cgroup.cfg
$ update-grub
$ reboot
```

## Run your first cluster

**Create Cluster**

The next step is to create a configuration. TPA does most of the work for you by way of its configure command. All you have to do is supply command-line flags and options to select, in broad terms, what you want to deploy. Here's the tpaexec configure command:

```
tpaexec configure workshop --architecture M1 --platform docker --postgresql 15 --enable-repmgr --no-git
```

**Provision Cluster**

Now you're ready to create the containers (or virtual machines) on which to run the new deployment. Use the provision command to achieve this:

```tpaexec provision workshop```
You will see TPA work through the various operations needed to prepare to deploy your configuration.

**Deploy Cluster**

Once the containers are provisioned, you can move on to deployment. Deploying installs, if needed, operating systems and system packages. It then installs the requested Postgres architecture and performs all the needed configuration.

```tpaexec deploy workshop```
You will see TPA work through the various operations needed to deploy your configuration.


**Test Cluster**

You can quickly test your newly deployed configuration using the tpaexec test command. This command runs pgbench on your new database.

```tpaexec test workshop```

## Docker Container Management 

All of the docker containers in a cluster can be started and stopped together using the start-containers and stop-containers commands:

```
[tpa]$ tpaexec start-containers workshop
[tpa]$ tpaexec stop-containers workshop
```
These commands don't provision or deprovision containers, or even connect to them; they are intended to save resources when you're temporarily not using a docker cluster that you need to keep available for future use.

For a summary of the provisioned docker containers in a cluster, whether started or stopped, use the list-containers command:

```
[tpa]$ tpaexec list-containers workshop
```
