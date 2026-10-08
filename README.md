# ansible_code
ansible to setup Backoffice server configurations

# Install Ansible inside WSL

sudo apt update
sudo apt install -y python3-venv openssh-client

mkdir -p ~/ansible-ec2
cd ~/ansible-ec2

python3 -m venv .venv
source .venv/bin/activate

python -m pip install ansible-core

ansible --version

wsl --set-default Ubuntu-24.04
wsl -d Ubuntu-24.04

# create a dedicated Ansible SSH key

ssh-keygen -t ed25519 -f ~/.ssh/ansible_ec2 -C "ansible-controller"

~/.ssh/ansible_ec2       ← private key: stays on your laptop
~/.ssh/ansible_ec2.pub   ← public key: install on EC2

# Add it to the public key EC2 instance

whoami

mkdir -p ~/.ssh
chmod 700 ~/.ssh

touch ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys

nano ~/.ssh/authorized_keys

# Create inventory.ini file to make ssh connection

nano inventory.ini

[ec2_servers]
ec2_1 ansible_host=EC2_1_PUBLIC_IP
ec2_2 ansible_host=EC2_2_PUBLIC_IP

[ec2_servers:vars]
ansible_user=ubuntu
ansible_port=22
ansible_ssh_private_key_file=~/.ssh/ansible_ec2
ansible_python_interpreter=/usr/bin/python3

#  Confirm SSH works, replacing the IP placeholders

ssh -i ~/.ssh/ansible_ec2 ubuntu@EC2_1_PUBLIC_IP 'whoami'
ssh -i ~/.ssh/ansible_ec2 ubuntu@EC2_2_PUBLIC_IP 'whoami'

# test Ansible and install on each server

ansible ec2_servers -i inventory.ini -m ansible.builtin.ping

ansible-playbook -i inventory.ini install-packages.yml --syntax-check

ansible-playbook -i inventory.ini install-packages.yml --limit ec2_1
ansible-playbook -i inventory.ini install-packages.yml --limit ec2_2