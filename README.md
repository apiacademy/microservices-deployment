# ansible-consul-demo

Demo of Automated Microservices Infrastructure Setup using Ansible, Docker, Consul and the friends. Demo is tested and uses AWS cloud, but can be adapted for other clouds.

## Installation

1. Install Ansible
2. Install and configure Boto:
    1. Installation: https://boto3.readthedocs.org/en/latest/guide/quickstart.html#installation
    
       On OS-X El Capitan, you will have to ignore `six` that is pre-installed and causes issues:
    
       ```console
       sudo -H pip install --ignore-installed boto3
       ```
    
    2. Configuration: https://boto3.readthedocs.org/en/latest/guide/quickstart.html#configuration

1. Spin-up some Ubuntu servers on AWS (or any other hosting)
1. Edit the IPs of the servrs in the provided `hosts` file (present at the 
same level as this README). Please make sure to indicate **public** IPs.
1. Make sure to also edit `group_vars/all.yml` and enter the IP(s) of all servers
   that you allocated as consul servers. The entries in hosts and group_vars.all.yml
   must correspond to each other!
1. While you are in `group_vars/all.yml` you may also want to change the `shell_user`
   variable. By default it is set to "ubuntu" because that's what AWS wants to log
   into your Ubuntu servers in as, but another user may make sense to you.
1. In AWS EC2, the root username for Ubuntu servers is called: `ubuntu`. If you 
   spin your servers up somewhere where that is not the case, edit 
   `group_vars/all.yml` and modify the value of the `ansible_ssh_user: ubuntu` variable.
1. Create `ssh` folder under this checkout (same level as README)
1. Save a private SSH key that corresponds to the root user on all your new servers
   under: `ssh/private-key.pem`
1. Make sure your private key permissions are valid:
       
       ```consul
       chmod 700 ssh
       chmod 600 ssh/*
       ```
1. You are probably going to need following ports open (based on 
   <https://www.consul.io/docs/agent/options.html> customize as you see fit):
    ![](http://media.froyo.io/image/402Y3I2o393G/Configuration_-_Consul_by_HashiCorp.png)
1. If you need to copy security group configuration across AWS regions, this script is a life-saver: https://github.com/pedropregueiro/migrate-ec2-secgroups    

## Quickstart

To make sure your ssh and hosts setup is correct and you can login to all 
required servers:

```console
ansible all -m ping -i hosts
```

To install basic Linux tools (curl, vim etc.) on all servers:

```console
ansible-playbook basics.yml -i hosts
```

To install consul server and clients:

```console
ansible-playbook bootstrap.yml -i hosts
```

To install everything, including the sample "hello world" microservice in Node.js:

```console
ansible-playbook webheads.yml -i hosts
```

## Consul User Interface

```
http://<ip-of-a-consul-server>:8500/
```

Your microservices will be available at: http://microservice-hello.service.consul on any host that points to Consul DNS servers as the DNS.

## Debugging

Consul logs are under: `/var/log/upstart/consul.log`

To see current members of Consul cluster: 

```
consul members
```

To make sure that consul leadership election succeeded (bootstrapping),
you can run the following on a consul server:

```
consul info
```

and analyze the `raft:` section of the response.
## Troubleshooting

If you are on a network that doesn't allow access to custom port you can create an SSH proxy:

```
ssh -D 12345 myuser@remote_ssh_server
```

and then in your browser proxu settings indicate SOCKS5 proxy with hostname: localhost, port: 12345.
