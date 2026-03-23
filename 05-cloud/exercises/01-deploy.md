# Exercise: Deploying dockerized application on multiple VMs

In this exercise, we will deploy a dockerized application on multiple VMs.

You should have two VMs:
- `1GB RAM / 1 CPU`: This VM will run the backend and frontend containers.
- `2GB RAM / 1 CPU`: This VM will run the database container.

We will use a [sample application](https://github.com/ek-osnb/cloud-starter), that has:
- A Spring Boot application that connects to a MySQL database.
- A frontend Single page application (with client side routing) that connects to the Spring Boot application.
- A MySQL database that stores the data.

For simplicity, we will run both the spring boot app and the frontend app on the same VM, and the database on the other VM.


## Step 0: Fork and clone the repository
For this to work, you need to have your own copy of the [sample application](https://github.com/ek-osnb/cloud-starter). You can achieve this by forking the repository and then cloning it to your local machine.

Open the repository up in your VS Code, and take a look at the `compose.prod.app.yml`.

You need to replace the image names in the `compose.prod.app.yml` file to point to your own GHCR repository. The image names should be in the following format:
```bash
ghcr.io/GITHUB_USERNAME/deploy-test/app:latest
```
Make sure to replace `GITHUB_USERNAME` with your actual GitHub username. This will ensure that when we run the application, it will pull the images from your GHCR repository.

**Commit and push the changes to your forked repository on GitHub.**


# Step 1: Install Docker on both VMs

Install Docker on both VMs by following the instructions in the docker installation guide. You can use the same installation script for both VMs. Make sure to run the script on both VMs to install Docker.

Make sure to test that Docker is installed correctly on both VMs by running `sudo systemctl status docker` and checking that the Docker service is active and running.


## Step 2: Create a Release on GitHub
Before we can deploy our application, we need to create a release on GitHub. There is a workflow that automatically builds and pushes the Docker images to GitHub Container Registry (GHCR) when a new release is created.

To create a release, head to the "Releases" section of your forked repository on GitHub and click on "Draft a new release". Give your release a tag (e.g., `v0.0.1`) and a title, then click on "Publish release".


## Step 3: Initial Setup
ssh into the DB VM, and create a new directory called `db`:

```bash
mkdir db
```
Then, download the `compose.prod.db.yml` file over to the `db` directory:

```bash
wget https://raw.githubusercontent.com/ek-osnb/cloud-starter/refs/heads/main/compose.prod.db.yml -O db/compose.prod.db.yml
```

This will download the `compose.prod.db.yml` file from the GitHub repository and save it in the `db` directory. This file contains the Docker Compose configuration for the MySQL database.

Download the sample `.env` file and save it in the `db` directory:

```bash
wget https://raw.githubusercontent.com/ek-osnb/cloud-starter/refs/heads/main/.env -O db/.env
```

This will download the `.env` file from the GitHub repository and save it in the `db` directory. This file contains the environment variables for the MySQL database.

> Change the `MYSQL_ROOT_PASSWORD` variable in the `.env` file to a strong password of your choice. This password will be used to access the MySQL database.


## Step 4: Exposing the database to the VPC

For security reasons, we will not expose the database to the public internet. Instead, we will only allow access to the database from other VMs in the same VPC. To do this, we will need to modify the `.env` file and set the `MYSQL_PRIVATE_IP` variable to only be exposed to the private IP addresses of the VMs in the same VPC.

On DigitalOcean, you can find the private IP address of your VM in the "Droplets" section of the dashboard. Click on your VM and look for the "Private IP" in the header section. 

Copy the private IP address of the VM and set the `MYSQL_PRIVATE_IP` variable in the `.env` file to the following:

```bash
MYSQL_PRIVATE_IP=REPLACE_WITH_PRIVATE_IP
```

> Replace `REPLACE_WITH_PRIVATE_IP` with the actual private IP address of the VM. Typically the private IP address will be in the format `10.x.x.x` or `192.168.x.x`.


## Step 5: Creating a Personal Access Token (PAT) for GHCR
To pull the Docker images from GitHub Container Registry (GHCR), we need to authenticate with GHCR using a Personal Access Token (PAT). 

1. Go to [https://github.com/settings/tokens](https://github.com/settings/tokens) and click **Generate new token** use the **classic** option.
2. Give your token a name and select the following scopes:
	- `read:packages` (to pull images)
3. Generate the token and **copy it somewhere safe** (you won't be able to see it again).

## Step 6: Authenticate with GHCR and run the database container
You need to authenticate with GHCR on the DB VM using the PAT you just created:

```bash
echo YOUR_PAT | docker login ghcr.io -u YOUR_GITHUB_USERNAME --password-stdin
```
Replace `YOUR_PAT` and `YOUR_GITHUB_USERNAME` with your actual values.

This command will log you in to GHCR using your GitHub username and the PAT you created. This will allow you to pull the Docker images from GHCR.


Now that we have the Docker Compose file and the environment variables set up, we can run the database container on the DB VM. Navigate to the `db` directory and run the following command:

```bash
docker compose -f compose.prod.db.yml up -d
```

This command will start the MySQL database container in detached mode (running in the background). The `-f` flag specifies the Docker Compose file to use.

You can check that the container is running by using the following command:

```bash
docker ps -a
```

## Step 7: Running the application

Now that the database is running, you can exit the db VM (type `exit` in the terminal). Then, ssh into the app VM:

```bash
ssh root@APP_VM_IP_ADDRESS
```
Replace `APP_VM_IP_ADDRESS` with the actual public IP address of the app VM.


Create a new directory called `app`:

```bash
mkdir app
```
Then, navigate to the `app` directory 

```bash
cd app
```

And download the `compose.prod.app.yml` file from **your forked repository**:

```bash
wget https://raw.githubusercontent.com/YOUR_GITHUB_USERNAME/cloud-starter/refs/heads/main/compose.prod.app.yml -O compose.prod.app.yml
```
Replace `YOUR_GITHUB_USERNAME` with your actual GitHub username. This ensures you get the version with your own GHCR image names that you updated in Step 0.

Also download the sample `.env` file and save it in the `app` directory:

```bash
wget https://raw.githubusercontent.com/ek-osnb/cloud-starter/refs/heads/main/.env -O .env
```
This file contains the environment variables for the Spring Boot application and the frontend application.

> Change the `SPRING_DATASOURCE_URL` variable in the `.env` file to point to the private IP address of the DB VM. The format should be as follows:
```bash
SPRING_DATASOURCE_URL=jdbc:mysql://DB_VM_PRIVATE_IP:3306/mydb
```
Replace `DB_VM_PRIVATE_IP` with the actual private IP address of the DB VM.

Authenticate with GHCR on the app VM using the same command as before:

```bash
echo YOUR_PAT | docker login ghcr.io -u YOUR_GITHUB_USERNAME --password-stdin
```
Replace `YOUR_PAT` and `YOUR_GITHUB_USERNAME` with your actual values.

Now that we have the Docker Compose file and the environment variables set up, we can run the application containers on the app VM. Run the following command:

```bash
docker compose -f compose.prod.app.yml up -d
```
This command will start the Spring Boot application and the frontend application containers in detached mode (running in the background). The `-f` flag specifies the Docker Compose file to use.

You can check that the containers are running by using the following command:

```bash
docker ps -a
```

## Step 8: Test the application
To test that the application is running correctly, open a web browser and navigate to the public IP address of the app VM. You should see the frontend application running. You can also test that the frontend application can communicate with the Spring Boot application and the database by creating a new item in the frontend application and checking that it is stored in the database.

## Step 9: Setting up Firewall Rules with UFW


In this exercise, we will set up firewall rules on our VMs to enhance security. We will use UFW (Uncomplicated Firewall) to manage the firewall rules.

Firewalls are essential for protecting your server from unauthorized access and potential attacks. UFW is a user-friendly interface for managing firewall rules on Linux systems. It allows you to easily allow or deny incoming and outgoing traffic based on specific criteria.

Our goal is to have the following rules:
- **app server**: Allow incoming traffic on ports 80 (HTTP) and 443 (HTTPS) from any IP address, and allow SSH on port 22 from any IP address (to avoid locking ourselves out of the VM).
- **db server**: Allow incoming traffic on port 3306 (MySQL) only from the private IP address of the app server, and allow SSH on port 22 from any IP address (to avoid locking ourselves out of the VM).

### APP SERVER

SSH into the app server and follow the steps to set up UFW rules as described in the previous exercise, but with the rules specific to the app server.

#### Step 1: Install UFW

ufw should be installed by default on the VM, but if it is not, you can install it using the following command:

```bash
sudo apt update
sudo apt install ufw
```

#### Step 2: Status of UFW
You can check the status of UFW with the following command:

```bash
sudo ufw status
```
It should show that UFW is inactive.

#### Step 3: Setting up UFW rules

As a best practice, we will set the default policies to deny all incoming traffic and allow all outgoing traffic:

```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing
```

We can then specify which ports we want to allow. We want to be able to:
- Access our application on port 80 (HTTP) and 443 (HTTPS) from any IP address.
- Access SSH on port 22 from any IP address (to avoid locking ourselves out of the VM).

To allow incoming traffic on these ports, run the following commands:

```bash
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
sudo ufw allow 22/tcp
```
**If we didn't allow port 22, we would lock ourselves out of the VM! So it's important to allow this port.**

#### Step 4: Enable UFW
Now that we have set up our rules, we can enable UFW:

```bash
sudo ufw enable
```

Confirm the changes by checking the status again:

```bash
sudo ufw status
```

You should see that ports 22, 80, and 443 are allowed.

#### Step 5: Test access
Before you exit your terminal session, it's a good idea to test that you can still access the VM via SSH. Open a new terminal window and try to SSH into the VM:

```bash
ssh root@VM_IP_ADDRESS
```
If you can successfully connect, then the firewall rules are working correctly. If you cannot connect, you may need to review the UFW rules and ensure that port 22 is allowed.

### DB SERVER

SSH into the db server and follow the steps to set up UFW rules as described in the previous exercise, but with the rules specific to the db server.

#### Step 1: Install UFW

ufw should be installed by default on the VM, but if it is not, you can install it using the following command:

```bash
sudo apt update
sudo apt install ufw
```

#### Step 2: Status of UFW
You can check the status of UFW with the following command:

```bash
sudo ufw status
```
It should show that UFW is inactive.

#### Step 3: Setting up UFW rules
As a best practice, we will set the default policies to deny all incoming traffic and allow all outgoing traffic:

```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing
```

We can then specify which ports we want to allow. We want to be able to:
- Access MySQL on port 3306 only from the private IP address of the app server
- Access SSH on port 22 from any IP address (to avoid locking ourselves out of the VM).

To allow incoming traffic on these ports, run the following commands:

```bash
sudo ufw allow from APP_SERVER_PRIVATE_IP to any port 3306 proto tcp
sudo ufw allow 22/tcp
```
Replace `APP_SERVER_PRIVATE_IP` with the actual private IP address of the app server.

**If we didn't allow port 22, we would lock ourselves out of the VM! So it's important to allow this port.**

#### Step 4: Enable UFW
Now that we have set up our rules, we can enable UFW:

```bash
sudo ufw enable
```

Confirm the changes by checking the status again:

```bash
sudo ufw status
```

You should see that ports 22, and 3306 (with the specific IP address) are allowed.

#### Step 5: Test access
Before you exit your terminal session, it's a good idea to test that you can still access the VM via SSH. Open a new terminal window and try to SSH into the VM:

```bash
ssh root@VM_IP_ADDRESS
```
If you can successfully connect, then the firewall rules are working correctly. If you cannot connect, you may need to review the UFW rules and ensure that port 22 is allowed.