# V-Server Setup

Use this guide to set up a Linux V-Server with SSH key authentication, disabled password login, NGINX, and GitHub SSH access.

## Table of Contents

1. [Prerequisites](#prerequisites)
2. [Create a Local SSH Key](#create-a-local-ssh-key)
3. [Copy the Public Key to the Server](#copy-the-public-key-to-the-server)
4. [Verify SSH Login](#verify-ssh-login)
5. [Disable Password Login](#disable-password-login)
6. [Install NGINX](#install-nginx)
7. [Serve an Alternative NGINX Page on Port 8081](#serve-an-alternative-nginx-page-on-port-8081)
8. [Configure Git on the Server](#configure-git-on-the-server)
9. [Create a GitHub SSH Key on the Server](#create-a-github-ssh-key-on-the-server)
10. [Configure SSH for GitHub](#configure-ssh-for-github)
11. [Verify the Setup](#verify-the-setup)
12. [Checklist](#checklist)

## Prerequisites

- A running Ubuntu server
- A local terminal with SSH installed
- A GitHub account
- `sudo` access on the server

## Create a Local SSH Key

Generate a dedicated SSH key pair on your local machine.

```bash
ssh-keygen -t ed25519 -C "<your_email>" -f ~/.ssh/id_ed25519_vserver
```

Display the public key if you want to inspect it before copying it to the server.

```bash
cat ~/.ssh/id_ed25519_vserver.pub
```

## Copy the Public Key to the Server

Copy the public key from your local machine to the server.

```bash
ssh-copy-id <your_username>@<your_server_ip>
```

If you use a non-default key name, specify it explicitly.

```bash
ssh-copy-id -i ~/.ssh/id_ed25519_vserver.pub <your_username>@<your_server_ip>
```

## Verify SSH Login

Connect to the server with your private key.

```bash
ssh -i ~/.ssh/id_ed25519_vserver <your_username>@<your_server_ip>
```

The login should work without asking for the server password.

## Disable Password Login

Open the SSH server configuration on the server.

```bash
sudo nano /etc/ssh/sshd_config
```

Set the following values.

```text
PubkeyAuthentication yes
PasswordAuthentication no
KbdInteractiveAuthentication no
PermitRootLogin no
```

Validate the SSH configuration.

```bash
sudo sshd -t
```

Restart the SSH service.

```bash
sudo systemctl restart ssh
```

Test that password-based login is disabled from your local machine.

```bash
ssh -o PubkeyAuthentication=no <your_username>@<your_server_ip>
```

The command should fail with `Permission denied (publickey)`.

## Install NGINX

Update the package list on the server.

```bash
sudo apt update
```

Install NGINX.

```bash
sudo apt install -y nginx
```

Enable the service.

```bash
sudo systemctl enable nginx
```

Start the service.

```bash
sudo systemctl start nginx
```

Check the service status.

```bash
systemctl status nginx
```

## Serve an Alternative NGINX Page on Port 8081

Create a directory for the alternative page.

```bash
sudo mkdir -p /var/www/vserver-demo
```

Create the HTML file for the alternative page.

```bash
sudo nano /var/www/vserver-demo/index.html
```

Use this content.

```html
<!doctype html>
<html lang="en">
  <head>
    <meta charset="utf-8">
    <title>V-Server Demo</title>
  </head>
  <body>
    <h1>V-Server Demo Page</h1>
    <p>This page is served by NGINX on port 8081.</p>
  </body>
</html>
```

Create a dedicated NGINX server block.

```bash
sudo nano /etc/nginx/sites-available/vserver-demo
```

Use this configuration.

```nginx
server {
    listen 8081;
    listen [::]:8081;

    server_name _;
    root /var/www/vserver-demo;
    index index.html;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

Enable the new site.

```bash
sudo ln -s /etc/nginx/sites-available/vserver-demo /etc/nginx/sites-enabled/vserver-demo
```

Test the NGINX configuration.

```bash
sudo nginx -t
```

Reload NGINX.

```bash
sudo systemctl reload nginx
```

Open the page in a browser.

```text
http://<your_server_ip>:8081
```

## Configure Git on the Server

Set the Git user name on the server.

```bash
git config --global user.name "<your_name>"
```

Set the Git email address on the server.

```bash
git config --global user.email "<your_email>"
```

Review the active Git configuration.

```bash
git config --global --list
```

## Create a GitHub SSH Key on the Server

Generate a dedicated SSH key pair on the server for GitHub access.

```bash
ssh-keygen -t ed25519 -C "<your_email>" -f ~/.ssh/id_ed25519_github
```

Display the public key.

```bash
cat ~/.ssh/id_ed25519_github.pub
```

Add the public key to your GitHub account under `Settings -> SSH and GPG keys`.

## Configure SSH for GitHub

Create the SSH config file on the server.

```bash
nano ~/.ssh/config
```

Use this configuration.

```sshconfig
Host github.com
  HostName github.com
  User git
  IdentityFile ~/.ssh/id_ed25519_github
  IdentitiesOnly yes
```

Set the correct permissions.

```bash
chmod 600 ~/.ssh/config
```

Test the GitHub SSH connection.

```bash
ssh -T git@github.com
```

GitHub should confirm successful authentication.

## Verify the Setup

Verify that SSH key login works.

```bash
ssh -i ~/.ssh/id_ed25519_vserver <your_username>@<your_server_ip>
```

Verify that password login is disabled.

```bash
ssh -o PubkeyAuthentication=no <your_username>@<your_server_ip>
```

Verify that NGINX is running.

```bash
systemctl status nginx
```

Verify that the alternative page is reachable.

```text
http://<your_server_ip>:8081
```

Verify GitHub SSH access.

```bash
ssh -T git@github.com
```

## Checklist

- [x] Create an SSH key
- [x] Copy the public key to the server
- [x] Verify SSH login with the key
- [x] Disable password-based SSH login
- [x] Install and start NGINX
- [x] Serve an alternative NGINX page on port `8081`
- [x] Configure Git on the server
- [x] Create a dedicated GitHub SSH key on the server
- [x] Configure SSH for GitHub
- [x] Verify the complete setup
