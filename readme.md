# Assignment 2

## Overview
In this assignment, we will use **Terraform** for provisioning our infrastructure and **Ansible** for configuring each instance to handle different services.

## Prerequisites
- Terraform installed ([Download & Install Terraform](https://www.terraform.io/downloads.html))
- Ansible installed ([Installation Guide](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html))
- AWS CLI configured with appropriate permissions

## Steps

### Provision Infrastructure with Terraform
Navigate to the `infrastructure` directory where the `main.tf` file is located. This `main.tf` file will create the required cloud infrastructure components.

```sh
cd infrastructure
terraform init
terraform plan
terraform apply
enter value: yes
```
![Alt text](image-1.png)

#### This main.tf will create:
                              - an VPC

                              - two subnets (public and private)

                              - an Internet gateway

                              - a routing table with associations

                              - two security groups (public and subnet) with 
                              necessary inbound and ountbound rules

                              - 3 ec2 intsances (db backend web for different service)

                              - also very important it will create inventoy.yml file in the ../service/inventory folder which we will use for configuration our instances by anisble-playbook

                              - also ../service/group_vars/variables.yml which we will use to configure oue template files in /service/template folder.

                              - it also output the variables what we have in output.tf
![Alt text](image2.png)


### Configuration with Ansible
Navigate to the `service` directory where all ansible-play book files are locate and the `ansinle.cfg` file which will:
                                        
                                        - look in the ./inventory directory for the inventory file.

                                        - enable the aws_ec2 plugin for the inventory
#### Template
All files in this folder will use for ansible configures in `j2` format

#### group_vars
- variables.yml: store all 3 public dns name for 3 ec2 intances we created which will be used to configure in ansible-play book

```
 vars_files:
    - ./group_vars/variables.yml

```
#### inventory
- inventory.yml: stored all 3 publid dns host name used for playbok
```
hosts: all
hosts: db
hosts: backend
hosts: web

```


```sh
cd ../service
ansible-playbook -i ./inventory/inventory.yml basic_host_setup.yml
ansible-playbook -i ./inventory/inventory.yml db_host_setup.yml
ansible-playbook -i ./inventory/inventory.yml backend_host_setup.yml
ansible-playbook -i ./inventory/inventory.yml web_host_setup.yml
```

`basic_host_setup.yml`:
                        1. this file will Setup basic host functionality and troubleshooting tools: all hosts

                        - update all OS package list

                        - installed packages : bind9-dnsutils tcpdump nmap mysql-client

`db_host_setup.yml`:
                    1. Installation and Configuration of Database Server(database instance)

                    - Install mysql-server and python3-pymysql

                    - Configure mysql to listen on the private IP, rewrite bind-address

                    - Enable and restart mysql service

                    - Remove anonymous user accounts ''

                    - Remove the 'test' database

                    - Create application database

                    - Create application user and grant all privileges on the database

                    - Create table for the application and insert initial data

                    - Remove remote root login

                    - Set the MySQL root password

`backend_host_setup.yml`:
                        1. Config Backend App: backend server(backend instance)

                        - Install necessary packages - git, libmysqlclient-dev,  pkg-config, python3-dev, python3-pip

                        - Create a group for the application

                        - Create an OS user for the application and add user to the new group

                        - Clone application code from repository

                        - Copy application files to appropriate locations

                        - Ensure backend user owns its home directory

                        - Install Python dependency packages

                        - Configure backend via config file in ./tmeplate folder: (backend.conf.j2)

                        - Create systemd service file from template (example.service.j2)

                        - Enable and restart the example service

`web_host_setup.yml`:
                    1. Config Web Server: web server(web instance)
                    
                    - Install Nginx and Git

                    - Create web root directory in /var/www

                    - configure Nginx using file in template folder: nginx.conf.j2

                    - Clone the repository with front end code

                    - Copy web files to the new web root location

                    - restart nginx service
