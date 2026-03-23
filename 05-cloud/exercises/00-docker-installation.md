# Docker setup on Virtual Machines

In this exercise, we will install Docker on our VM to be able to run docker containers.

## Step 1: Docker installation script

We can follow the official Docker installation guide for [Ubuntu](https://docs.docker.com/engine/install/ubuntu/#install-using-the-repository) to install Docker.

To make the installation easier, we will use a convenience script, which is basically the same commands as in the official guide, but wrapped in a script (although it's recommended to follow the official guide as the script may not always be up to date).

On your VM, run `vim install-docker.sh` or `nano install-docker.sh` to create a new script file called `install-docker.sh` and paste the following content into it:

```bash
#!/bin/bash

# Add Docker's official GPG key:
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update

# Install the latest version
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y
```

When you have pasted the content, save and exit the file (in `vim`, you can do this by pressing `Esc`, then typing `:wq` and pressing `Enter`).

## Step 2: Run the installation script

To make the script executable, run the following command:

```bash
chmod 700 install-docker.sh
```
This will give the script file the necessary permissions to be executed.

Now, run the script:

```bash
./install-docker.sh
```
This will execute the script and install Docker on your VM. The installation process may take a few minutes to complete, as it needs to download the necessary packages and set up Docker on your system.

When the installation is complete, check that Docker is installed correctly by running:

```bash
sudo systemctl status docker
```
You should see that the Docker service is active and running.