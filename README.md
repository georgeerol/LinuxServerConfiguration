# Linux Server Configuration

This project is about deploying a flask application on to AWS Light Sail. It provides the steps on creating an instance and deploy a flask application.

* AWS IP Address: http://18.215.148.81/
* AWS Port: 2200

## Get Started on Lighsail
Go to Amazon Lightsail to setup a server instance.
## Setup the Server
1. Login
2. Create an instance
3. Choose Ubuntu(OS only) as the instance image
4. Choose an instance plan
5. Give the instance a hostname

## Update the Server
Update all installed packages with these commands:
```sh
sudo apt-get update
sudo apt-get upgrade
```
Set auto upgrade to low unattted upgrades via these commands:
```sh
sudo dpkg-reconfigure --priority=low unattended-upgrades
```

## Create a new user
Create the grader or a new user with this command:
```sh
sudo adduser grader
```
After creating the user run this command to give her/him sudo privilidges:
```sh
sudo visudo
```
Once in the visudo file, under section user privilege specification  under root add `grader  ALL=(ALL:ALL) ALL`
It should look like this:
```sh
root    ALL=(ALL:ALL) ALL
grader  ALL=(ALL:ALL) ALL
```
To logins as the grader,use this command:
```
sudo login grader
```
# Configure SSH to non-default port
From AWS LightSail Firewall page, add a custom application port 2200.


