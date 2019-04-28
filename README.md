# Linux Server Configuration

This project is about deploying a flask application on to AWS LIght Sail. It provides the steps on creating an instance and deploy a flask application.

* AWS IP Address: http://18.215.148.81/
* AWS Port: 2200

## Get Started on Lighsail
Go to Amazon Lightsail to setup a server instance.
### Setup the Server
1. Login
2. Create an instance
3. Choose Ubuntu(OS only) as the instance image
4. Choose an instance plan
5. Give the instance a hostname

### Secure the Server
Update all installed packages with these commands:
```sh
sudo apt-get update
sudo apt-get upgrade
```
