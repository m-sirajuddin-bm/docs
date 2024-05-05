# How to Create SSH Key in Digital Ocean Droplet

This guide will show you how to create SSH private and public keys and use it in github actions for deployment or for any other use case. I am using Ubuntu version 22.04 for this.

## Login to Server

Login to the using below command and enter the server password.

```bash
ssh root@xxx.xx.xx.xx
```

## Generate SSH Keys

```bash
ssh-keygen -t rsa
```

It will ask you to enter file in which to save the key (/root/.ssh/id_rsa). It usually go with the logged in user file. 

> NOTE: I am using **root** user to generate the keys here so that it will not prompt to enter a password when any command is triggered from your application using this ssh key.

Enter the default name or you can give any name.

```bash
Enter file in which to save the key (/root/.ssh/id_rsa): /root/.ssh/id_ed25519
```

> NOTE: By default, SSH searches for id_rsa, id_ecdsa, id_ecdsa_sk, id_ed25519, id_ed25519_sk, and id_dsa files. The keys do not have to be named like this, you can name it 'mykey' just as well, or even place it in a different directory.

It will then prompt to enter the passphrase for your SSH key. If you are using this key for deployment in github actions or for some automation it is better to leave it empty so that it will not prompt to enter the passphrase during automation.

Press enter twice to continue and the following response will be show up on successful creation of the keys.

```bash
Your identification has been saved in /home/username/.ssh/id_ed25519
Your public key has been saved in /home/username/.ssh/id_ed25519.pub
The key fingerprint is:
a9:49:EX:AM:PL:E3:3e:a9:de:4e:77:11:58:b6:90:26 username@203.0.113.0
The key's randomart image is:
+--[ RSA 2048]----+
|     ..o         |
|   E o= .        |
|    o. o         |
|        ..       |
|      ..S        |
|     o o.        |
|   =o.+.         |
|. =++..          |
|o=++.            |
+-----------------+
```

To view the keys generated, type:

```bash
# For private key
cat ~/.ssh/id_ed25519

# Private will look like below:

# -----BEGIN OPENSSH PRIVATE KEY-----
# MIIEowIBAAKCAQEAmtpWciLisnyhX18kIYrkWPJmXcosOu96GXe+aiAbH1RYXatu
# .....
# .....
# l0X9oTpeL8gL8I9MVFvpgCuzkeF6ff9PHHp6ynGAipJg70yf8uO2==
# -----END OPENSSH PRIVATE KEY-----


#For public key
cat ~/.ssh/id_ed25519.pub

# Public will look like below:

#ssh-rsa EXAMPLEzaC1yc2E...GvaQ== username@203.0.113.0
```

You can save the generated private key, public key and fingerprint in a safe place.

# Upload SSH Public Key to Droplet

The public keys listed in "~/.ssh/authorized_keys" are the ones that you can use to log in to the server as this user, so you need to add the public key you copied into this file.

To do so, run the following command on your Droplet, replacing the example key in quotes (ssh-rsa EXAMPLEzaC1yc2E...GvaQ== username@203.0.113.0) with the key you copied:

Copy the public key output, which will look similar to this example:

```bash
echo "ssh-rsa EXAMPLEzaC1yc2E...GvaQ== username@203.0.113.0" >> ~/.ssh/authorized_keys
```

Alternatively, you can open the ~/.ssh/authorized_keys file with a terminal-based text editor, like nano, and paste the contents of the key into the file that way.

The ~/.ssh directory and authorized_keys file must have specific restricted permissions (700 for ~/.ssh and 600 for authorized_keys). If they don’t, you won’t be able to log in (non root users).

Once the authorized_keys file contains the public key, set the permissions and ownership of the files:

```bash
chmod -R go= ~/.ssh
chown -R $USER:$USER ~/.ssh
```

# Useful links

The following links helped to create this document:

- [Adding SSH Keys](https://docs.digitalocean.com/products/droplets/how-to/add-ssh-keys/)
- [Adding SSH Keys to an Existing Droplet](https://docs.digitalocean.com/products/droplets/how-to/add-ssh-keys/to-existing-droplet/)
- [Adding SSH Keys to a Team](https://docs.digitalocean.com/products/droplets/how-to/add-ssh-keys/to-team/)
