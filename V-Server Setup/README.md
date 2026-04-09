# V-Server Setup

This document describes how I configured my V-Server for the Developer Akademie DevSecOps course project.

## Table of Contents

1. [Project Overview](#project-overview)
2. [Server Information](#server-information)
3. [SSH Setup](#ssh-setup)
4. [Disable Password Login](#disable-password-login)
5. [NGINX Installation and Configuration](#nginx-installation-and-configuration)
6. [Git Configuration on the Server](#git-configuration-on-the-server)
7. [GitHub SSH Key Setup](#github-ssh-key-setup)
8. [Testing](#testing)
9. [Conclusion](#conclusion)

## Project Overview

The goal of this project was to set up a secure Linux V-Server, configure SSH key authentication, disable password-based SSH login, install and configure NGINX, and prepare Git and GitHub access directly on the server.

## Server Information

- Server IP: `91.98.162.80`
- Server user: `fatih-yalcin`
- Operating system: Ubuntu 24.04 LTS

## SSH Setup

First, I copied my public SSH key from my local machine to the server.

```bash
ssh-copy-id -i /Users/fatihyalcin/Desktop/DevSec/ssh-key/demo_ed25519.pub fatih-yalcin@91.98.162.80


Then I tested the login with my private key:
ssh -i /Users/fatihyalcin/Desktop/DevSec/ssh-key/demo_ed25519 fatih-yalcin@91.98.162.80
This confirmed that SSH key authentication was working correctly.


Disable Password Login
After verifying that SSH login with the key worked, I updated the SSH server configuration:
sudo nano /etc/ssh/sshd_config

I used the following settings:
PubkeyAuthentication yes
PasswordAuthentication no
KbdInteractiveAuthentication no
PermitRootLogin no

Then I validated the configuration:
sudo sshd -t

And restarted the SSH service:
sudo systemctl restart ssh

To make sure password-based login was really disabled, I tested it from my local machine:
ssh -o PubkeyAuthentication=no fatih-yalcin@91.98.162.80

This login failed with Permission denied (publickey), which confirmed that password login was disabled.
NGINX Installation and Configuration

I installed NGINX on the server using apt:
sudo apt update
sudo apt install -y nginx
sudo systemctl enable nginx
sudo systemctl start nginx

Then I checked the service status:
systemctl status nginx

After that, I verified in the browser that the default NGINX page was reachable via:
http://91.98.162.80

Next, I replaced the default landing page with a custom HTML page:
sudo nano /var/www/html/index.nginx-debian.html

After editing the file, I tested and reloaded the NGINX configuration:
sudo nginx -t
sudo systemctl reload nginx
This successfully displayed my custom landing page in the browser.

Git Configuration on the Server
I configured Git on the server with my name and GitHub email address:
git config --global user.name "Fatih Yalcin"
git config --global user.email "86c29vxm24@privaterelay.appleid.com"
git config --global --list
This ensures that commits made on the server use the correct identity.

GitHub SSH Key Setup
To allow GitHub access directly from the server, I generated a dedicated SSH key pair on the server:
ssh-keygen -t ed25519 -C "86c29vxm24@privaterelay.appleid.com" -f ~/.ssh/id_ed25519_github
Then I displayed the public key:
cat ~/.ssh/id_ed25519_github.pub
I added this public key to my GitHub account under:

Settings
SSH and GPG keys
New SSH key
After that, I created an SSH config file on the server:
nano ~/.ssh/config
With this content:
Host github.com
  HostName github.com
  User git
  IdentityFile ~/.ssh/id_ed25519_github
  IdentitiesOnly yes
Then I set the correct file permissions:
chmod 600 ~/.ssh/config
Finally, I tested the GitHub SSH connection:
ssh -T git@github.com
GitHub returned a successful authentication message.

Testing
I verified the following points:

SSH login with key authentication works
SSH login with username and password is disabled
NGINX is installed and running
The custom landing page is reachable in the browser
Git is configured with the correct user name and email
GitHub SSH authentication from the server works

Conclusion
This project helped me understand the basic setup and security configuration of a Linux V-Server.

I learned how to:

configure SSH key authentication
disable password-based SSH login
install and configure NGINX
deploy a custom HTML landing page
configure Git on a server
authenticate a server with GitHub via SSH

Sensitive data such as passwords and private SSH keys are not included in this repository.

