# Ubuntu Server Preparation

> For cloud ubuntu 18.04

## SSH
1. Check ssh server
```
sudo systemctl status ssh
```

2. Check ip `ip a`
3. Generate qif rsa
```
ssh-keygen -t rsa
```

4. Copy
```
pbcopy < ~/.ssh/id_rsa.pub
```

5. Paste to Server
```
cat > authorized_keys
粘贴
```

6. Change Mode `chmod 400 authorized_keys`
7. Change config
```
Host qif-prod
  HostName 192.168.26.99
  AddKeysToAgent yes
  UseKeychain yes
  User qifadmin
  IdentityFile ~/.ssh/qif-prod
```

## Set password for sudo

**Important**
```
$ sudo passwd root
```


## Docker
1.Install Docker to Ubuntu

```ssh
lsb_release -a
```

```sh
# Tested in Ubuntu 16.04 & 18.04
sudo apt-get update
sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    software-properties-common
    
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -

sudo apt-key fingerprint 0EBFCD88
sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"
   
```

- if seen **Malformed input, repository not added.**

```
# Do
sudo add-apt-repository    "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   Ubuntu Bionic \
   stable"
# 

sudo vi /etc/apt/sources.list.d/additional-repositories.list
# Change into below
deb [arch=amd64] https://download.docker.com/linux/ubuntu bionic stable
```

```
sudo apt-get update
sudo apt-get install docker-ce
```

Test
```
sudo docker run hello-world
```

## Install Docker Compose
[Install Docker Compose](https://docs.docker.com/compose/install/)
```
sudo curl -L "https://github.com/docker/compose/releases/download/1.24.1/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

sudo chmod +x /usr/local/bin/docker-compose
```

**Manage Docker as a non-root user* [Manage Docker as a non-root user](https://docs.docker.com/install/linux/linux-postinstall/)
```
Create the docker group.

$ sudo groupadd docker
Add your user to the docker group.

$ sudo usermod -aG docker $USER
Log out and log back in so that your group membership is re-evaluated.

If testing on a virtual machine, it may be necessary to restart the virtual machine for changes to take effect.

On a desktop Linux environment such as X Windows, log out of your session completely and then log back in.

On Linux, you can also run the following command to activate the changes to groups:

$ newgrp docker 
Verify that you can run docker commands without sudo.

$ docker run hello-world
```

## Github

1. Copy git key to server
```
scp  ~/.ssh/id_rsa_git qif-prod:~/.ssh
```

1. Edit config

```
cd ~/.ssh
cat > ~/.ssh/config <<EOF
Host github.com
 AddKeysToAgent yes
 IdentityFile ~/.ssh/id_rsa_git
EOF
```

## Clone without password required
```
git clone git@github.com:chunkai-meng/qif.git
```