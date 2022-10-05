# Installing Confluent by CP Ansible

The project contain 4 use case of installing confluent platform.
We simulate a VM host with docker image of centos7, and we build a custom image with pre-installing some mandatory package(java, open-ssl....) to save time during installation.

development : base installation without encryption or security
tls-enabled : Encryption using custom certifications
tls-plain : Encryption using custom certifications, and SASL PLAIN for authentication
mtls : Using mTLS for authentication

## Setup Ansible

Steps to configure ansible env ( tested on WSL2) :

- Install python3

- Create a new Python virtual environment: `python3 -m venv confluent-with-ansible`

- Activate a Python virtual environment: `source confluent-with-ansible/bin/activate`

- Install Ansible in a virtual environment: `python3 -m pip install ansible-core==2.11.12` 

- Install requirements:  `ansible-galaxy install -r requirements.yml`

## 1 Build the custom image for confluent components

`docker build --rm -t local/centos7-efg .`

## 2 Start all the components of our infra

`docker-compose up -d`

## 2a (optional) Generate custom certification (for tls-enabled & tls-plain)

`ansible-playbook certs/certificates.yml -i inventories/tls-enabled/development.yml`

A custom certifications will be generated at certs/generated_ssl_files

## 3 Installing confluent platform with Ansible

`ansible-galaxy collection install confluent.platform:7.2.1`

## 4 go with development (it takes time):

`ansible-playbook -i inventories/development/development.yml confluent.platform.all`

## 4a (optional) tls-enabled:

`ansible-playbook -i inventories/tls-enabled/development.yml confluent.platform.all --skip-tags validate_ssl_keys_certs`

## 4b (optional) tls-plain:

`ansible-playbook -i inventories/tls-plain/development.yml confluent.platform.all --skip-tags validate_ssl_keys_certs`

## 5 available tasks
`ansible-playbook -i inventories/development/development.yml confluent.platform.all --list-tasks`
