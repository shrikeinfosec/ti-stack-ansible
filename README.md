# MISP Stack with Ansible

This repo provides a baseline template for deploying [MISP](https://github.com/MISP/MISP) and [Cortex](https://github.com/TheHive-Project/Cortex) to a newly-provisioned Ubuntu 22.04 machine (Jammy).

> [!note]
> This is a work-in-progress repository, and should not be considered production-safe for all environments. 
> 
> These configurations may not be sufficiently hardened for your environment.

# Pre-Requisites

This guide assumes that you already have Python v3 and Ansible installed.
See ['Installing and upgrading Ansible'](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html#installing-and-upgrading-ansible) for more specific installation information.

> **Warning**
> 
> This playbook can ONLY be run from a Linux-based machine. It is not possible to use a Windows machine as a control node to execute these commands. Please ensure these files are present on your control node and run the below command. 

You will need to install the following on your control node:

- The Docker community modules via ansible-galaxy (Allows for Ansible to handle the creation of the Docker containers)

```bash
ansible-galaxy collection install community.docker
```
- sshpass (Adds support for providing SSH password during playbook run)

```bash
sudo apt-get install sshpass
```

# Configuration
There are a few included files that can be customised to better suit your environment. These are:

## .template.env

This file contains the primary configuration information for your MISP and Cortex stack. You'll need to provide the relevant values for your environment and save it as `.env` into the same directory as the `playbook.yaml` file.

## playbook.yaml

This is the primary Ansible playbook file. You can customise the steps for the installation of Docker to better match your environment (e.g., if you are running a different version of Ubuntu, you can change references to `jammy` to the relevant match for your OS.)

The playbook is designed for Debian-based distributions, so you will almost certainly run into issues if you're trying to run it out-of-the-box on anything else.


---

# Usage

From your control node, run the following command:

```bash
ansible-playbook playbook.yaml --extra-vars "target=<hostname.local>" --ask-pass --ask-become-pass -e 'ansible_python_interpreter=/usr/bin/python3'
```

## Parameters

> `--extra-vars "target=<hostname.local>"`

Where `<hostname.local>` is your FQDN for your target node. This allows you to specify the host you want to target.

> [!note]
>
> This does not bypass the requirement of having your host listed in an Ansible inventory. 
> 
> If you do not have one (or are unsure as to what that is), create a file called `hosts` at the following location: `/etc/ansible/` and add the following:
> - `hostname.local`
>
> For more information, see ['Building Ansible inventories'](https://docs.ansible.com/ansible/latest/inventory_guide/index.html).

---
> `--ask-pass`

Asks for the SSH password so that it does not need to be stored in the playbook file.

---

> `--ask-become-pass`

Asks for the password to become root via `sudo` (for the target machine).

---

> `-e 'ansible_python_interpreter=/usr/bin/python3'`

This specifies that the Python3 interpreter is used when running commands from the control node.

Once you have run this command, Ansible will run through the process of provisioning the machine.

> [!note]
>
> This can take up to 30 minutes to complete, as Docker will need to download all of the base images and run the install scripts for MISP. Ansible will most likely appear stuck at the 'Run MISP Docker container' stage while this happens.

Once the process is complete, you should be able to navigate to `https://<hostname.local>` / `https://<hostname-ip>` to access the MISP application and `https://<hostname.local>:9001` / `https://<hostname-ip>:9001` to access the Cortex application (provided you haven't adjusted the configuration files).

# Important Information

The Ansible Docker modules will use the `docker-compose` package for handling the compose files. As such, when you need to stop or start a container on the target machine, you will need to ensure you use `sudo docker-compose` command and **not** `sudo docker compose`.

Using `docker compose` will cause Docker to get confused over the ownership of the container and may cause unexpected conflicts when starting/stopping containers.

To stop a container, use:
```bash
sudo docker-compose down
```

To start a container (in detached mode), use:
```bash
sudo docker-compose up -d
```

To run these commands from your control node, use:
```bash
ansible <hostname.local> -a "<command_to_run>" --ask-pass --ask-become-pass
```