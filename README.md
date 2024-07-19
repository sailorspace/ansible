# ansible
module -> smallest task that you have 
inventory -> information about tasks in the servers 
playbook -> task that you want to perform 

Main Server(101-Orca) Ansible -> Access other servers(remote) without password 
                             #-> ssh-keygen //use this to generate a keygen 
                             #-> ssh-copy-id 192.168.211.102 //to copy the id to remoteserver
							 #-> ssh-copy-id vagrant@192.168.211.102 //for using current user as permissioned to remote
							     //mostly this is the case, create a new ssh-keygen for each user access 
						     
                              -> this will ask the password for one time to set things in remote server(102 here)
                              -> the key/certificate generated is copied to the remote server 
                              -> now ssh from main server to any machine will work without asking password 
							  
- make sure your user has the acces rights and permissions to run commands
  for vagrant give the user root priviledges
  open file in the target machine where you want to run the tasks from ansible master machine e.g 192.168.211.102
  - sudo su   //enter the target machine root terminal first 
  - sudo -i  //
  - passwd  //call for change password 
  - vi /etc/ssh/sshd_config //enter the input mode by pressing "i"
    - PermitRootLogin yes  //uncomment this and change to this 
	- PasswordAuthentication yes //make this change as well 
    - press excape and :wq
 - service ssh restart   //restart the service 

# Installing Ansible :
https://docs.ansible.com/ansible/latest/installation_guide/installation_distros.html#installing-ansible-on-ubuntu

- sudo apt update
- sudo apt install software-properties-common
- sudo add-apt-repository --yes --update ppa:ansible/ansible
- sudo apt install ansible


config file = /etc/ansible/ansible.cfg

- ansible --version //check the installed version 
- ansible localhost -m ping //test if ansible is working or not 
- goto /etc/ansible and check the defaut config file , it will be empty with directions to create sample config 
- ansible-config init --disabled > ansible.cfg //an example ansible config created with all settings disabled
  //on ansible installation the config file is empty without any settings
- mkdir playbooks -> cd playbooks  //playbok can be made at any directory
- nano first-pb.yml //create a playbook 
  vi <filename>.yml //to enter in insert mode press "i"
#sample
  ---
  - name: this is first playbook 
    hosts: localhost //where to run the task..could be remote server, this mean we are pointing the localhost 
                    //in the hostfile 	
	tasks: //section start for all the task(s) 
	- name: install package   //describing the intent of the task being said
	  apt:                  //this is the module that is doing the task, apt is ubuntu module to run packages  
	   name: nginx         //give a package name that you want to run 
	   state: present     //ask ansible to make sure the package is presetn after installation 
	- name: start the service 
	  service: 
	  name: nginx     //give the service name on which we need to do some action
	  state: started //make sure the service is started 
	  enabled: true //enable the service 

-#ansible-playbook <playbook file path> //to run the playbook 
- tasks run in order in ansible 
-#ansible-playbook --syntax-check <playbook-file-name> //to check the syntax of the yml file before running it
-# service nginx status //check if the service is started, e.g for the nginx here 
-# systemctl status nginx.service //anotherway to check the status 
--------------------------------------------------------------------------------
https://docs.ansible.com/ansible/2.9/modules/list_of_all_modules.html
e.g https://docs.ansible.com/ansible/2.9/modules/service_module.html#service-module
	
- go to the ansible config directory /etc/ansible/
- there will be a host file just as the windows
- map the remote server ip here where we want to run a task action 
- the hosts file has some commented examples of how to target actions as per the hosts 
- hosts can be grouped and ungrouped 
- if we want to run task(s) on only a set of ips that belong to a group e.g web server then we can 
- provide the name of the group , rather then putting same task for each of the ips, reducing time and effort
- vi hosts // in the ungrouped section add the server ip where you would run the tasks
- add the new inventory i.e new ip address of the remote
  e.g for vagrant based machines 
   192.168.211.102 ansible_ssh_pass=vagrant ansible_ssh_user=vagrant
   //for other machines with root password we will need to add only the ip address 
-#ansible-inverntory --list //command to check validate our inventory details
-#ansible all -m ping //after the above changes, ansible will look for all the IP(s)/servers and run the ping tasks       
                     //on them, this is a way to check if the ansible master maschine is able to contact the remote
- Now that we have added the grouped and ungrouped servers 
- we will update the playbook to run tasks on remote servers 
- in the playbook we wil need to update the target ips or groups. 
- "all" use all for running the playbook tasks on all the machines and groups mentioned in playbook 
- Applying the playbook on specific target 
  #somefile.tml 
  ---
  - name: app install playbook
    hosts: 192.168.211.102  //could be "all" or any group name 

    tasks:
    - name: installation of nginx package
      apt:
        name: nginx
        state: present
    - name: starting the nginx service
      service:
       name: nginx
       state: started
       enabled: true

- validate both the playbook and hosts inventory again 
  - ansible-playbook --syntax-check <filename.yml>  //check if the playbook file is all good 
  - ansible-inventory --list   //check i hosts file is all good
  - ansible all -m ping       //check if the target machines are all good 
  - ansible 192.168..211.102 -m ping //check if our specific target is all good with ansible making contact
  - finally run the playbook ansible-playbook <playbook>.yml 
- when there is any change in runnning the playbook the changes will be shown

#Ad-Hoc commands in ansible
- ansible <host-pattern> -m <modulename> -a <module arguments> 
     - e.g ansible localhost -m ping
        - ansible all -m ping 
	- ansible 192.168.211.102 -m ping
	- ansible webserver -m ping //pinging all the servers under the group webserver
	- ansible 192.168.211.102 -m copy -a "src=/home/vagrant/wick.txt dest=/temp/scripts/"
	- ansible 192.168.211.102 -m copy -a "src=/temp/test.txt dest=/temp/script/" -b //for any user not root -b mean become priviledged
	- ansible all -m service -a "name=nginx state=reloaded"
	- ansible all -m shell -a "/temp/scripts/test.sh"
	 	
	
#Tagging in playbook tasks 
- ansible-playbook tag-multi-tasks.yml --list-tags //this lists all the tags in the playbook file
- ansible-playbook tag-multi-tasks.yml -t inginx  //run a specific task based on the tag marking 
- ansible-playbook tag-multi-tasks.yml --skip-tags <tag-name-toskip> //this will skip all the tags mentioned and run others

#if any value needs hashing or some function controlled 
-  password: "{{'pass123' | password_hash('sha512') }}"  #password needs to be hashed, we cant provide the text format

#variables declaration 
---
  - name: this is first playbook
    hosts: 192.168.211.102
    vars:
     - app: nginx

    tasks:
    - name: install package
      apt:
       name: "{{app}}"
       state: present
      tags: in-nginx          # adding a tag so that we run run tag scoped playbook not every task

    - name: start the service
      service:
       name: "{{app}}"
       state: started
       enabled: true
      tags: ss-nginx        # tagging here to mark the niginx start and enablie

#declaring the hosts file IPs by name
- orca ansible_host=192.168.211.102 //defining by name so that we do not have to pass the ip all over again 
- ansible orca -m ping //this works with the server name in the inventory 

#handlers in ansible 
 tasks:
  - name: enable the service in the firewalld
    firewalld:
      port: 80/tcp
      permanent: true
      state: enabled   #enabled the service, any change in the fiewall needs reloading of the firewall
    notify:
      - reload firewalld  # this triggers the handlers i.e notifies the handler to run its tasks

  handlers:  # handlers are always triggerd when required
  - name: reload firewalld
    service:
       name:  firewalld
       state: reloaded

#conditions in ansible 
---
- name: handling conditions based on the platform
  hosts: all

  tasks:
  - name: install httpd on red hat
    yum:
     name: httpd
     state: present
    when: ansible_od_family == "RedHat"


  - name: install apche2 on ubuntu
    apt:
      name: apache2
      state: present
    when: ansible_os_family == "ubuntu"

 
- ansible localhost -m setup //protuces all the properties variables available for ansible
//gets all the modules and variable info 
- ansible orca -m setup     //getting all the built in variables for the remote server

#loops in ansible 
 tasks:
    - name: install package
      apt:
       name: "{{item}}"         # app changes to item here in the placeholder to replace each item in the loop
       state: present
      tags: set-one             # adding a tag so that we run run tag scoped playbook not every task
      with_items: "{{apps}}"    # provide the listname to choose the items from in ordet to iterate

 
#Ansible roles 
a structured way of grouping together various functionalities and making it reuse and share command tasks 
- ansible-galaxy init role-name  //to crate a role 
- if we go to the /etc/ansible we will find hosts,ansible.config and roles here 
- ansible-galaxy init apache2_setup //this will create a role under roles folder
- ls 
- this will have created some set of files under the role apache2_setup folder 
- cd apache2_setup
- ls -ltr 
	drwxr-xr-x 2 root root 4096 Jul 19 09:18 vars 
	drwxr-xr-x 2 root root 4096 Jul 19 09:18 templates
	-rw-r--r-- 1 root root 1328 Jul 19 09:18 README.md
	drwxr-xr-x 2 root root 4096 Jul 19 09:18 files
	drwxr-xr-x 2 root root 4096 Jul 19 09:18 tests
	drwxr-xr-x 2 root root 4096 Jul 19 09:18 tasks
	drwxr-xr-x 2 root root 4096 Jul 19 09:18 defaults
	drwxr-xr-x 2 root root 4096 Jul 19 09:18 meta
	drwxr-xr-x 2 root root 4096 Jul 19 09:18 handlers
- cd tasks //get inside the tasks folder 
https://cn-ansibledoc.readthedocs.io/zh-cn/latest/galaxy/user_guide.html




