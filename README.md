# ansible-config-mgt

In this project, we would work on Ansible Configuration Management where we would automate processes on other servers

It will make us appreciate DevOps tools even more by making most of the routine tasks automated with Ansible Configuration Management, at the same time, we will become confident at writing code using declarative language such as YAML.

##  Task

1. Install and configure Ansible client to act as a Jump Server/Bastion Host

2. Create a simple Ansible playbook to automate servers configuration

##  Step 1 - INSTALL AND CONFIGURE ANSIBLE ON EC2 INSTANCE

* Update Name tag on your Jenkins EC2 Instance to Jenkins-Ansible. We will use this server to run playbooks.

![Jenkins=Ansible](https://user-images.githubusercontent.com/117458922/230644839-0a9e842b-c541-448b-af69-eae91f1d67c5.png)

* In your GitHub account create a new repository and name it ansible-config-mgt.

![git-ansible](https://user-images.githubusercontent.com/117458922/230645282-651e93b1-a05a-4659-9699-310ae0f3aad6.png)

* Install ansible

```
sudo apt update
sudo apt install ansible
```

* Check your Ansible version by running:

```
ansible --version
```

![ansible version](https://user-images.githubusercontent.com/117458922/230645716-c2755842-306b-4840-8990-a488b1c2886a.png)

* Configure Jenkins build job to save your repository content every time you change it:

Create a new Freestyle project ansible in Jenkins and point it to your ‘ansible-config-mgt’ repository.

![configrepo](https://user-images.githubusercontent.com/117458922/230646607-f9d02c58-3195-4d31-be0c-7ed1ed81be24.png)


Configure Webhook in GitHub and set webhook to trigger ansible build.

![webhook](https://user-images.githubusercontent.com/117458922/230647369-f13eaa5b-5219-4e80-afca-f039d65ec878.png)

Configure a Post-build job to save all (**) files, like we did it in the Jenkins project

![post build](https://user-images.githubusercontent.com/117458922/230647697-c77adba2-9c1b-4b55-92d7-fc3456ae6d86.png)

Test your setup by making some change in README.MD file in master branch and make sure that builds starts automatically and Jenkins saves the files

![test build](https://user-images.githubusercontent.com/117458922/230647977-b9e8a3d2-d84a-43b4-b991-a0d3667f8e90.png)

Confirm if Jenkins saves the files (build artifacts) in following folder:

```
sudo ls /var/lib/jenkins/jobs/ansible/builds/<build_number>/archive/
```

![check connect](https://user-images.githubusercontent.com/117458922/230648905-ce0bed24-e28a-4112-b78b-eaa0ae788a32.png)

Now our setup will look like this:

![setup](https://user-images.githubusercontent.com/117458922/230649706-457a17f7-43d3-42bd-9726-12036f30c07c.png)


##  Step 2 – Prepare your development environment using Visual Studio Code


* First part of ‘DevOps’ is ‘Dev’, which means you will require to write some codes and you shall have proper tools that will make your coding and debugging comfortable – you need an Integrated development environment (IDE) or Source-code Editor. There is a plethora of different IDEs and Source-code Editors for different languages with their own advantages and drawbacks, you can choose whichever you are comfortable with, but we recommend one free and universal editor that will fully satisfy your needs – Visual Studio Code (VSC), you can get it here.

* After you have successfully installed VSC, configure it to connect to your newly created GitHub repository.

* Clone down your ansible-config-mgt repo to your Jenkins-Ansible instance


##  Step 3 - BEGIN ANSIBLE DEVELOPMENT

* In our ansible-config-mgt GitHub repository, we will create a new branch that will be used for development of a new feature. I named mine prj-11

```
git checkout -b prj-11
```

* Checkout the newly created feature branch to your local machine and start building your code and directory structure

* Create a directory and name it playbooks – it will be used to store all your playbook files.
* Create a directory and name it inventory – it will be used to keep your hosts organised.
* Within the playbooks folder, create your first playbook, and name it common.yml
* Within the inventory folder, create an inventory file (.yml) for each environment (Development, Staging Testing and Production) dev, staging, uat, and prod respectively.

![directories](https://user-images.githubusercontent.com/117458922/230650919-b6bf4539-062f-4819-a289-d3b4f09b7603.png)


##  Step 4 – Set up an Ansible Inventory

Ansible uses TCP port 22 by default, which means it needs to ssh into target servers from Jenkins-Ansible host – for this reason we need to copy our private (.pem) key to the server.

```
eval `ssh-agent -s`
ssh-add <path-to-private-key>
```

* Confirm the key has been added with the command below, you should see the name of your key

```
ssh-add -l
```

* Now, ssh into your Jenkins-Ansible server using ssh-agent

```
ssh -A ubuntu@public-ip
```

* We will update the inventory/dev.yml file with this snippet of code:

```
[nfs]
<NFS-Server-Private-IP-Address> ansible_ssh_user='ec2-user'

[webservers]
<Web-Server1-Private-IP-Address> ansible_ssh_user='ec2-user'
<Web-Server2-Private-IP-Address> ansible_ssh_user='ec2-user'

[db]
<Database-Private-IP-Address> ansible_ssh_user='ec2-user' 

[lb]
<Load-Balancer-Private-IP-Address> ansible_ssh_user='ubuntu'
```

![dev](https://user-images.githubusercontent.com/117458922/230651787-ee5c7e3b-9b0b-4880-9086-c6b75271fc08.png)


##  Step 5 – Create a Common Playbook

It is time to start giving Ansible the instructions on what we need it to be perform on all servers listed in inventory/dev.
In common.yml playbook we will write configuration for repeatable, re-usable, and multi-machine tasks that is common to systems within the infrastructure.

* Update the playbooks/common.yml file with following code:

```
------
- name: update web, nfs and db servers
  hosts: webservers, nfs, db
  remote_user: ec2-user
  become: yes
  become_user: root
  tasks:
  - name: ensure wireshark is at the latest version
    yum:
      name: wireshark
      state: latest

- name: update LB server
  hosts: lb, db2
  remote_user: ubuntu
  become: yes
  become_user: root
  tasks:
    - name: Update apt repo
      apt: 
        update_cache: yes

    - name: ensure wireshark is at the latest version
      apt:
        name: wireshark
        state: latest
```

![common](https://user-images.githubusercontent.com/117458922/230652460-14ed0345-131c-4bef-bfd3-01f5e64e4379.png)

##  Step 6 – Update GIT with the latest code

Now all of our directories and files live on your machine and we need to push changes made locally to GitHub.

* Commit your code into GitHub:

use git commands to add, commit and push your branch to GitHub.

```
git status

git add <selected files>

git commit -m "commit message"
```

![git](https://user-images.githubusercontent.com/117458922/230653885-d36d12c2-1c29-4c3b-8556-9196ea9d0745.png)

![commit](https://user-images.githubusercontent.com/117458922/230653543-75c748e2-5698-40be-90c7-3a1e6194938c.png)

![push](https://user-images.githubusercontent.com/117458922/230653587-18118815-912b-4b4b-b141-e7d8cb2ffca8.png)

* Create a Pull request (PR)

```
git pull
```

![pull](https://user-images.githubusercontent.com/117458922/230654175-341c8261-5e72-44b4-bd0e-73b48158e5b4.png)

* Head back on your terminal, checkout from the feature branch into the master, and pull down the latest changes.

Once your code changes appear in main branch – Jenkins will do its job and save all the files (build artifacts) to /var/lib/jenkins/jobs/ansible/builds/<build_number>/archive/ directory on Jenkins-Ansible server.

![jk jon](https://user-images.githubusercontent.com/117458922/230654441-a0ec0820-4289-4b51-be0e-dd69abd88172.png)


##  Step 7 – Run first Ansible test

Now, it is time to execute ansible-playbook command and verify if your playbook actually works:

```
ansible-playbook -i /var/lib/jenkins/jobs/ansible/builds/<build-number>/archive/inventory/dev.yml /var/lib/jenkins/jobs/ansible/builds/<build-number>/archive/playbooks/common.yml
```

![playbook](https://user-images.githubusercontent.com/117458922/230655223-fff4d78b-928b-4ae3-a456-98ea0d93e339.png)


You can go to each of the servers and check if wireshark has been installed by running which wireshark or wireshark --version

![wireshark](https://user-images.githubusercontent.com/117458922/230655299-39081cb7-b3ec-4356-8a91-85ed0e29cbbf.png)

Our updated setup with Ansible architecture now looks like this:

![set](https://user-images.githubusercontent.com/117458922/230655836-c55a2a63-816f-473e-8197-982dbffc57dd.png)

#### *Great Job Done Today*
