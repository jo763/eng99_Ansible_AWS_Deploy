
# Create an EC2 instance for the ansible controller
- Use standard settings
- SSH into controller

- Edit hosts
`cd /etc/ansible/hosts`
- Edit/replace hosts file with the following
```
[localhost]
localhost
```

# Create Playbook to create EC2 instance
- Install dependencies
```
sudo apt-get update
sudo apt-get
install software-properties-common
sudo apt-add-repository --yes --update ppa:ansible/ansible
sudo apt-get update
sudo apt-get install ansible
sudo apt-get install tree
```
- Install Python
```
sudo apt install python
sudo apt install python-pip -y
sudo pip install --upgrade pip
sudo pip install boto
sudo pip install botocore
sudo pip install boto3
```
- If there are issues, change to Python3 via `alias python=python3` and `alias pip=pip3` and repeat the previous steps

- Obtain pem file either making a new one or rsync/cp or just ctrl_c, ctrl-v into a nanoed file
```
cd ~/.ssh
sudo nano eng99.pem
```

- Create yaml file
```
mkdir -p AWS_Ansible/group_vars/all/
cd AWS_Ansible
touch playbook.yml
```
- Create access vault
```
ansible-vault create group_vars/all/pass.yml
```
- Enter access and secret keys into vault
- press "i" to be able to type in vim
```
ec2_access_key: <Your_Access_Key>                                     
ec2_secret_key: <Your_Secret_Key>
```
- To leave press "ESC", then "shift + :", then "W", "Q" "ENTER"
- Create playbook to make an EC2 instance
```
sudo nano playbook.yml
```
- Fill with the following code:

```
---

- name: Create EC2 Instance
  hosts: localhost
  gather_facts: yes
  connection: local
  become: true

  tasks:

    - name: Create EC2 Instance
      tags: eng99_joseph_ansible_nginx
      ec2:
        aws_access_key: {{ec2_access_key}}
        aws_secret_key: "{{ec2_secret_key}}"
        key_name: eng99
        image: ami-07d8796a2b0f8d29c
        instance_type: t2.micro
        vpc_subnet_id: subnet-069c6968e4a264460
        group_id: sg-09fc5560cc95e7c1c
        region: eu-west-1
        count: 1
        wait: yes
        assign_public_ip: yes

        instance_tags:
          Name: eng99_joseph_ansible_nginx
```
- Change the permission of the vault file
```
cd /home/vagrant/AWS_Ansible/group_vars/all
```
- Change the permission of the vault file
```
sudo chmod 666 pass.yml
```
- Navigate to the Ansible folder
```
cd /home/vagrant/AWS_Ansible
```
- Execute the script
```
sudo ansible-playbook playbook.yml --ask-vault-pass
```

# Working in the created instance

- Create dockerfile
`sudo nano dockerfile`
- Assuming the index (and css) file we wish to replace in the nginx file in already in the instance via git or rsync
- Enter the following with the following:
```
FROM nginx:1.11-alpine
COPY index.html /usr/share/nginx/html/index.html
COPY standard.css /usr/share/nginx/html/standard.css
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```
- Build the image
`docker build -t XYZ:latest .`
- Run the image on port 80
`docker run -d -p 80:80 xyz`
- The website should now be viewable on the website
