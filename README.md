# Ansible Tutorial

#1. SSH
```
sudo apt install openssh-server
```
#SSH Keys
```
ssh-keygen -t ed25519 -C "admin default"
cat ~/.ssh/id_ed25518.pub | ssh tertol@192.168.100.129 "mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys"
ssh-copy-id ~/.ssh/id_ed25519.pub 192.168.100.129
```
#Entering ssh passphrase once unless terminal closed
```
eval $(ssh-agent) #to check the ssh in background

ssh-add #it will asked for passphrase
```

#2 Git
```
git config --global user.name "tertolhumper"
git config --global user.email "tertolhumper@protonmail.com"

git clone #github 

sudo cat /root/.github_token
git remote set-url origin https://tertolhumper:github_tokengithub.com/tertolhumper/ansible_tutorial.git
git push origin main
```

#3. Ansible 
```
sudo apt install ansible
```
create an inventory
```
cat > ~/ansible_tutorial/inventory.ini << 'EOF'
[arch]
arch_vm ansible_host=192.168.100.123 ansible_port=7001 ansible_user=tertol

[rhel]
rhel_vm ansible_host=192.168.100.4 ansible_port=7001 ansible_user=tertol
EOF
```
create ansible cfg
```
cat > ~/ansible_tutorial/ansible.cfg << 'EOF'
[defaults]
inventory = inventory.ini
remote_user = tertol
private_key_file = ~/.ssh/ansible
interpreter_python = auto_silent
EOF
```
check ping of ansible 
```
ansible all -m ping
```
list all hosts in ansible
```
ansible all --list-hosts
```
gather info on ansible
```
ansible all -m gather_facts # on all
ansible all -m gather_facts --limit arch #specific server
```

#4. Ansible adhoc commands with elevated privileges
update arch
```
ansible arch -m pacman -a "update_cache=true" --become --ask-become-pass
```
update rhel
```
ansible rhel -m dnf -a "update_cache=true" --become --ask-become-pass 
```
update all servers if within the same debian family 
```
ansible all -m apt -a "update_cache=true" --become --ask-become-pass
````
Note: Be cautious on the commands as not all servers have the same distro family.
