# Setting up Firewall Rules with UFW

In this exercise, we will set up firewall rules on our VMs to enhance security. We will use UFW (Uncomplicated Firewall) to manage the firewall rules.

Firewalls are essential for protecting your server from unauthorized access and potential attacks. UFW is a user-friendly interface for managing firewall rules on Linux systems. It allows you to easily allow or deny incoming and outgoing traffic based on specific criteria.

## Step 1: Install UFW
ufw should be installed by default on the VM, but if it is not, you can install it using the following command:

```bash
sudo apt update
sudo apt install ufw
```

## Step 2: Status of UFW
You can check the status of UFW with the following command:

```bash
sudo ufw status
```
It should show that UFW is inactive.

## Step 3: Setting up UFW rules

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

## Step 4: Enable UFW
Now that we have set up our rules, we can enable UFW:

```bash
sudo ufw enable
```

Confirm the changes by checking the status again:

```bash
sudo ufw status
```

You should see that ports 22, 80, and 443 are allowed.

## Step 5: Test access
Before you exit your terminal session, it's a good idea to test that you can still access the VM via SSH. Open a new terminal window and try to SSH into the VM:

```bash
ssh dev@VM_IP_ADDRESS
```
If you can successfully connect, then the firewall rules are working correctly. If you cannot connect, you may need to review the UFW rules and ensure that port 22 is allowed.

