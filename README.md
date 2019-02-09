<p align="center">
  <img width="200" height="200" src="https://upload.wikimedia.org/wikipedia/commons/e/e1/North_Carolina_State_University_Athletic_logo.svg">
</p>

# HW2-DevOps
CSC-519 Spring 2019

Name: Sujal\
Unity Id: ssujal

## Completed Tasks:
:white_check_mark: Using ansible, be able to automatically configure a server running mattermost <br>

:white_check_mark: Installing Ubuntu Server 16.04 LTS <br>

:white_check_mark: Installing MySQL Database Server (5.7.x is recommended) <br>

:white_check_mark: Installing Mattermost Server <br>

:white_check_mark: Configuring Mattermost Server <br>

---

## [Installation guide](https://docs.mattermost.com/install/install-ubuntu-1604.html)

Follow steps to complete each of the task.

## Requirements:
vars.yml: 
> Consists of variables like name of db, root password, mattermost user, hostname, teamname, user's firstname

secret.yml:
> Consists of the username, password, server and port of the email sending service. (encrypted)
> Fill in your details in the following format.
```
---
username:
  xxxx
password:
  xxxx
server:
  xxxx
port:
  xxx

```
> Encrypt using ```ansible-vault encrpyt <filename>```

Once the configuration server and web-server are setup, configure the files inside the ansible-srv/ inside the configuration server.
```
ansible-playbook install_mysql.yml install_mattermost.yml conf_mm.yml -i inventory --ask-vault-pass
```
> Set a new vault password and confirm the same when prompted

## Creating your servers (source: course website by Dr.Parnin)

### The configuration server

Let's create a configuration server. This server will be using a "push-based model", where we will be sending configuration commands to other external servers. It also needs to be configured with ansible.

Create the Virtual Machine.

```bash
$ cd servers/ansible-srv
$ cat baker.yml
$ baker bake
```

You should see baker create the virtual machine.

```
✔ Running apt-get update on VM
✔ Preparing ansible
✔ Installing ansible
```

Verify that ansible was installed by running opunit `cd ../..`, then `opunit verify -i opunit_inventory.yml`.

### The web server

Let's create another virtual machine for the web server. 

```bash
$ cd servers/web-srv
$ cat baker.yml
$ baker bake
```

You should see baker create the virtual machine. Verify `baker ssh` works, then `exit` back to your host machine.

## Creating a connection between your servers

You need a way to automatically connect to your server without having to manually authenicate each connection. We will create a pair of public/private keys for [authentication through ssh](https://www.ssh.com/ssh/public-key-authentication#sec-Asymmetric-Cryptography-Algorithms).

From your host machine, create a new public/private key pair, running the following command, and hitting enter for the default prompts:

    ssh-keygen -t rsa -b 4096 -C "web-srv" -f web-srv


After generating the keys, you need to copy them onto the servers. The private key (*secret*, tell know one 🤐), needs to be sent over to the configuration server. The public key (🌐), needs to be sent over to the web-server.

#### Setting up the private key

One nice trick is to use a copy utility to copy a file into your copy/paste buffer:

* Mac: `pbcopy < web-srv`
* Windows: `clip < web-srv`
* Linux: [Check out one of these options](https://superuser.com/a/288333/862331).

Let's go to the ansible-srv (`cd servers/ansible-srv`, then `baker ssh`).

Using a file editor, paste and store your private key in a file:

```bash
ansible-srv $ vim ~/.ssh/web-srv
# Make sure key is not readable by others.
ansible-srv $ chmod 600 ~/.ssh/web-srv
# We're done here, go back to host
ansible-srv $ exit
```

#### Setting up the public key

Copy the web-srv.pub file from your host.

Go inside the web-srv.

Using a file editor, add the public key to the list of authorized keys:

```bash
web-srv $ vim ~/.ssh/authorized_keys`
web-srv $ exit
```

Take care not to delete other entries, which may affect your ability to use vagrant/Baker to ssh into the machine. You also do not need to change the permissions of the file.

#### Testing your connection/Errors

Inside the ansible-srv, test your connection between the servers:

    ssh -i ~/.ssh/web-srv vagrant@192.168.33.100

Note: You should also be able to make this same connection from your host machine, since you also have private key, locally.

If you see an error or prompt for a password, you have a problem with your key setup. Double check have pasted in the content in correctly (where you in insert mode in vim before pasting?). 

If you see this warning, you need to remember to perform the `chmod 600` in order to fix the permissions of the private key file:

```
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@         WARNING: UNPROTECTED PRIVATE KEY FILE!          @
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
Permissions 0664 for '/home/vagrant/.ssh/web-srv' are too open.
```

## Performing commands over ssh

Once you have established a ssh connection between two servers, you have achieved an important milestone in configuration management.

You can now perform ad-hoc commands and even execute scripts on machines without having to manually log-in.

For example, when you run this command.

    ssh -i ~/.ssh/web-srv vagrant@192.168.33.100 ls /

You can see the directory of the web-srv.


**3. ScreenCast**
---

* The link to screencast is [here](https://drive.google.com/open?id=1NVaGuj76ZmbMMl-a2iRbcWeScLnMKrud)
