# IAC with Ansible


### Let's create Vagrantfile to create Three VMs for Ansible architecture
#### Ansible controller and Ansible agents 

```

# -*- mode: ruby -*-
 # vi: set ft=ruby :
 
 # All Vagrant configuration is done below. The "2" in Vagrant.configure
 # configures the configuration version (we support older styles for
 # backwards compatibility). Please don't change it unless you know what
 
 # MULTI SERVER/VMs environment 
 #
 Vagrant.configure("2") do |config|
 # creating are Ansible controller
   config.vm.define "controller" do |controller|
     
    controller.vm.box = "bento/ubuntu-18.04"
    
    controller.vm.hostname = 'controller'
    
    controller.vm.network :private_network, ip: "192.168.33.12"
    
    # config.hostsupdater.aliases = ["development.controller"] 
    
   end 
 # creating first VM called web  
   config.vm.define "web" do |web|
     
     web.vm.box = "bento/ubuntu-18.04"
    # downloading ubuntu 18.04 image
 
     web.vm.hostname = 'web'
     # assigning host name to the VM
     
     web.vm.network :private_network, ip: "192.168.33.10"
     #   assigning private IP
     
     #config.hostsupdater.aliases = ["development.web"]
     # creating a link called development.web so we can access web page with this link instread of an IP   
         
   end
   
 # creating second VM called db
   config.vm.define "db" do |db|
     
     db.vm.box = "bento/ubuntu-18.04"
     
     db.vm.hostname = 'db'
     
     db.vm.network :private_network, ip: "192.168.33.11"
     
     #config.hostsupdater.aliases = ["development.db"]     
   end
 
 
 end
```

# Checking Vagrant Instances

To check if the vagrant servers work and have internet access, we'll individually SSH into them with `vagrant ssh instancename` and perform the `sudo apt-get update -y && sudo apt-get upgrade -y` command on each one (web, db, controller).<br>

# Installing Ansible

- To install ansible, first let's ssh into the ansible instance: `vagrant ssh controller`<br>
- Now we will perform the pre-requisites to installing ansible<br>
- `sudo apt-get install software-properties-common`
- `sudo apt-add-repository ppa:ansible/ansible`
- Now we can install ansible
- `sudo apt-get install ansible -y`
- To check if ansible is successfully installed
- `ansible --version`
- Now we'll install a package that will help us to visualise the local storage
- `sudo apt-get install tree`
- Now we'll test it out 
- `cd /etc/ansible/`
- `tree`
- We need to now edit the hosts file here to tell ansible which servers you want to connect to and how
- `echo -e "[web]\n192.168.33.10 ansible_ssh_pass=vagrant\n[db]\n192.168.33.11 ansible_ssh_pass=vagrant" | sudo tee --append /etc/ansible/hosts`
- This should be appended to the end of the hosts file:
```yaml
[web]
192.168.33.10 ansible_ssh_pass=vagrant
[db]
192.168.33.11 ansible_ssh_pass=vagrant
```
- Next we'll try to connect to the servers specified in the hosts file
- `ssh vagrant@192.168.33.10` #this is the private IP of the app, password is 'vagrant'
- This should successfully connect, and we'll just exit it afterwards
- `exit`
- We'll do the same thing for the db server
- `ssh vagrant@192.168.33.11` #this is the private IP of the db, password is 'vagrant'
- `exit`
- We can also ping multiple servers at once to check their connectivity:
- `ansible web,db -m ping`
- On top of that, we have functionality to copy files from the localhost into the remote servers
- Let's create a file in the home directory which we'll try to copy to all the attached servers afterwards
- `sudo nano ~/readme.md`
- (Write anything inside the file and save it)
- `ansible all -m copy -a "src=~/readme.md dest=~/README.md"`
- Now the file should be copied into the home directory of all the connected servers, you can check this by doing the following command:
- `ansible all -a "ls"`
- Which should list the local storage of each connected node

# Benefits of Ansible
- Don't need to manually SSH into each server to try to execute commands 
- The more servers there are, the more impossible it becomes to do everything manually (e.g. if you have 100 servers, how will you perform a command in all 100 of them without a controller like ansible?)
- Can use ansible to gather information about all the servers at once or individual ones
- Very simple and easy to set up (it took us about 10 minutes to install ansible, and 5 minutes to edit the hosts file)

# What is IaC

- Managing and provisioning of infrastructure through code instead of through manual processes with an emphasis on self-service
- Such as being able to execute commands from multiple servers at once through code with a controller such as Ansible in YAML 
- Results in consistency throughout your configuration because it's all coming from the same code using the controller, which minimises the risk of human error
overall 
- Results in financial savings for a business that implements IaC via increased productivity and makes it much easier to scale up and attach new servers to the controller

ansible all -m ping

# Ansible Playbooks
- Playbooks save time
- They are YAML/YAML files that are used to implement configuration management
- They are also reusable 

## Creating a playbook:
  - file_name.yaml/yml
  - Starts with 3 dashes
  - Use spaces for indentation
  - Check logs by using `gather_facts: yes`
  - To get admin access, use `become: true`

  ```yaml
  # YAML/YML file to create a playbook to configure nginx in our web server 
---
# starts with 3 dashes

# add the name of the host/instace/vm
-  hosts: web

# collect logs or gather facts
   gather_facts: yes

# we need admin access to install anything
   become: true

# add instructions - install nginx - within the web server
   tasks:
   -  name: Installing nginx for our app server
      apt: pkg=nginx state=present
# be mindful of indentation
```
<br>

### Next, we will make a yml file to move the app folder into the web server, install all the dependencies required and then install and run the application:
```yaml
---

-  hosts: web
   gather_facts: yes
   become: true
   tasks:
   -  name: moving app folder in
      synchronize:
        src: /home/vagrant/app
        dest: ~/
   -  name: load a specific version of nodejs
      shell: curl -sL https://deb.nodesource.com/setup_6.x | sudo -E bash -

   -  name: install the required packages
      apt:
        pkg:
          - nginx
          - nodejs
          - npm
        update_cache: yes
   -  name: install and run the app
      shell:
         cd app; npm install; screen -d -m npm start
```