# Ansible Installation and Server Setup

This guide provides steps to install Ansible on an Ubuntu or CentOS-based system and set up servers for Ansible management with passwordless SSH access using a `.pem` key file.

## Ansible Installation

### For Ubuntu

1. **Add Ansible PPA Repository**:
    ```
    sudo apt-add-repository ppa:ansible/ansible
    ```

2. **Update the Package Index**:
    ```
    sudo apt update
    ```

3. **Install Ansible**:
    ```
    sudo apt install ansible
    ```

### For CentOS / linux

1. **Install EPEL Repository**:
    ```
    sudo yum install epel-release
    ```

2. **Install Ansible**:
    ```
    sudo yum install ansible
    ```

### Verify Ansible Installation same command for both

Check if Ansible is installed correctly:
```
ansible --version
```

## Configure Ansible Hosts

1. **Locate the Hosts File**:
    ```
    sudo vi /etc/ansible/hosts
    ```

2. **Edit the Hosts File**:

    Add your server details in the following format:

    ```
    [servers]
    server1 ansible_host=<Public IP>
    server2 ansible_host=<Public IP>
    server3 ansible_host=<Public IP>

    [servers:vars]
    ansible_python_interpreter=/usr/bin/python3
    ansible_user=ubuntu
    ansible_ssh_private_key_file=/home/ubuntu/keys/example.pem
    ```

3. **Create a Directory for Keys**:
    ```
    mkdir keys
    cd keys
    ```

4. **Get the Path to the Keys Directory**:
    ```
    pwd
    ```
    This should output something like `/home/ubuntu/keys`.

## Transfer the PEM Key File to the Ansible Master Node

From your local system (not EC2), transfer the `.pem` key file to the Ansible master node:

1. **Send PEM Key File to EC2 Ansible Master**:
    ```
    scp -i "example.pem" example.pem ubuntu@<ansible-master-public-ip>:/home/ubuntu/keys
    ```
2. **Verify the Key is Present on the Ansible Master**:
    ```
    cd /home/ubuntu/keys
    ls
    ```
3. **Set Correct Permissions for the Key (ie in your ansible master)**:
    ```
    chmod 600 /home/ubuntu/keys/example.pem
    ```

4. **Copy the Path to the Key File**:
    Use `pwd` to get the full path (e.g., `/home/ubuntu/keys/example.pem`) and update the `ansible_ssh_private_key_file` field in your hosts file accordingly.

## Final Hosts File Example

Your hosts file should look similar to this:

```
[servers]
server1 ansible_host=<Public IP>
server2 ansible_host=<Public IP>

[servers:vars]
ansible_python_interpreter=/usr/bin/python3
ansible_user=ubuntu
ansible_ssh_private_key_file=/home/ubuntu/keys/example.pem
```

## Test Passwordless Access via Ansible

Test if Ansible can access the servers using the `.pem` key file:

```
ansible servers -m ping
```

If everything is set up correctly, you should see a success message for each server similar to below.
```
ubuntu@ip:~$ ansible servers -m ping
server2 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
server1 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```
### Now create the Playbook File

Create a new YAML file for the playbook, for example, `install_nginx.yml`: you can use 'vi' or 'nano' 

```
- name: Install Nginx on servers
  hosts: servers
  become: yes
  tasks:
    - name: Ensure Nginx is installed (Ubuntu)
      apt:
        name: nginx
        state: present
      when: ansible_os_family == "Debian"

    - name: Ensure Nginx is installed (CentOS)
      yum:
        name: nginx
        state: present
      when: ansible_os_family == "RedHat"

    - name: Start and enable Nginx service
      service:
        name: nginx
        state: started
        enabled: yes
```
### Run the Ansible Playbook

Execute the playbook to install Nginx on your remote servers:

```
ansible-playbook -i /etc/ansible/hosts install_nginx.yml
```

### Step 4: Verify Nginx Installation

After running the playbook, verify that Nginx is installed and running on your servers.

1. **SSH into the Server**:
    ```
    ssh -i /home/ubuntu/keys/rock.pem ec2-user@<server-public-ip>
    ```

2. **Check Nginx Status**:
    ```
    sudo systemctl status nginx
    ```
3. **You can also Check Nginx Status by going to ec2 server ip example '1.1.1.1'**:
-  nginx page should open if it works fine




