# Microservices Demo

Demo of Automated Microservices Infrastructure Setup using Ansible, Docker, Consul and the friends. Demo is tested and uses AWS cloud, but can be adapted for other clouds.

## Installation

1. Install Ansible
2. Install and configure Boto:
    1. Installation: http://boto.readthedocs.org/en/latest/getting_started.html
    
       On OS-X El Capitan, you will have to ignore `six` that is pre-installed and causes issues:
    
       ```console
       sudo -H pip install --ignore-installed boto
       ```
    
    2. Configuration of Boto/AWS credentials: http://boto.readthedocs.org/en/latest/getting_started.html#configuring-boto-credentials
    
    3. Please note that there's currently a [major bug](https://github.com/ansible/ansible/issues/12883) in Ansible/`epc_vpc` which ignores profile parameter and uses default credentials, so you need to make sure that your **default** AWS credentials point to the environment where you need things created. 
    
3. Clone this repo with: 

    ```console
    git clone https://github.com/apiacademy/microservices-deployment.git
    ```
4. Go into the cloned repo and run the included [ec2 script](http://docs.ansible.com/ansible/intro_dynamic_inventory.html#example-aws-ec2-external-inventory-script), to make sure you properly installed and configured Boto:

    ```console
    ./ec2.py --list --boto-profile irakli-aws
    ```
    
    Make sure to replace `irakli-aws` with your AWS profile name from `~/.aws/credentials`. You can omit the option if you are using default profile.

1. Save a private SSH key that you use/will use the root user on your AWS servers under: `ssh/private-key.pem`. For security reasons, `ssh` folder is .gitignore-d in the demo repo, and you should ignore it, as well, if you build on top of the demo. NEVER check-in SSH keys into repos!

    **ATTENTION:** corresponding public key must be placed under `roles/aws-server-creation/ssh/public-key.pem`!!!

1. Make sure your private key permissions are valid:

    ```consul
    chmod 700 ssh
    chmod 600 ssh/*    
    ```
1. If you have some EC2 servers tagged with [key=Name, value=cademo_consuls], and the AWS credentials profile you are using is still `irakli-aws, then you can ping the tagged servers from Ansible with a command like:

    ```
    AWS_PROFILE=irakli-aws ansible -i ec2.py tag_Name_cademo_consuls -m ping    
    ```
    
    You can find more information about ec2 script and using dynamic EC2 inventories at: <https://aws.amazon.com/blogs/apn/getting-started-with-ansible-and-dynamic-amazon-ec2-inventory-management/>

1. Please make sure you set up proper AWS Profile in `group_vars/all.yml` under `aws_profile` variabe and then you can create required servers as easily as running:

```
ansible-playbook play-00-create-servers.yml -i hosts
```
       
## Quickstart

To make sure your ssh and hosts setup is correct and you can login to all 
required servers:

```console
ansible all -m ping -i hosts
```

To install basic Linux tools (curl, vim etc.) on all servers:

```console
AWS_PROFILE=irakli-aws ansible-playbook -i ec2.py play-baseline.yml
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
