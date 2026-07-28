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
## create an inventory
```
cat > ~/ansible_tutorial/inventory.ini << 'EOF'
[arch]
arch_vm ansible_host=192.168.100.123 ansible_port=7001 ansible_user=tertol

[rhel]
rhel_vm ansible_host=192.168.100.4 ansible_port=7001 ansible_user=tertol

[all:vars]
ansible_ssh_private_key_file=~/.ssh/ansible
ansible_python_interpreter=auto_silent
EOF
```
## check ping of ansible 
```
ansible -i inventory.ini all -m ping
```
