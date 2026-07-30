# Ansible Tutorial

#1. SSH
```
sudo apt install openssh-server
```
#SSH Keys
```
ssh-keygen -t ed25519 -C "admin default"
cat ~/.ssh/id_ed25519.pub | ssh tertol@192.168.100.129 "mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys"
ssh-copy-id ~/.ssh/id_ed25519.pub 192.168.100.129
```
#Entering ssh passphrase once unless terminal closed
```
eval $(ssh-agent) #to check the ssh in background

ssh-add #it will asked for passphrase
```

#2 Git
```
create a repo on your github ansible_tutorial
git clone it 

git config --global user.name "tertolhumper"
git config --global user.email "tertolhumper@protonmail.com"

sudo cat /root/.github_token
git remote set-url origin https://tertolhumper:github_token@github.com/tertolhumper/ansible_tutorial.git
git push origin main
```
Note: The git remote URL above uses a PAT (Personal Access Token) instead of a password as GitHub dropped password auth. If running as root, retrieve the token with `sudo cat /root/.github_token`. If running as a regular user, store your token somewhere accessible like `~/.github_token` and use `cat ~/.github_token` instead.

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
update, upgrade and install
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
upgrade all packages in rhel
```
ansible rhel -m dnf -a "name='*' state=latest" --become --ask-become-pass
```

install package in arch
```
ansible arch -m pacman -a name=vim --become --ask-become-pass
```
Note: Be cautious on the commands as not all servers have the same distro family.

#5. Ansible Playbook
Commands
```
ansible-playbook --ask-become-pass install_nginx.yml #install
ansible-playbook --ask-become-pass remove_nginx.yml 
```
Ansible package installation
```
cat > ~/ansible_tutorial/install_nginx.yml << 'EOF'
---
- name: Install nginx
  hosts: all
  become: true
  tasks:
    - name: Install nginx (Arch)
      community.general.pacman:
        name: nginx
        state: latest
      when: ansible_os_family == "Archlinux"

    - name: Install nginx (RHEL)
      ansible.builtin.dnf:
        name: nginx
        state: latest
      when: ansible_os_family == "RedHat"

    - name: Allow http through firewalld (RHEL)
      ansible.posix.firewalld:
        service: http
        permanent: true
        state: enabled
        immediate: true
      when: ansible_os_family == "RedHat"

    - name: Enable and start nginx
      ansible.builtin.service:
        name: nginx
        state: started
        enabled: true
EOF
```
Note on RHEL firewall 
```
sudo firewall-cmd --add-service=http --permanent && sudo firewall-cmd --reload
```

Ansible package removal
```
cat > ~/ansible_tutorial/remove_nginx.yml << 'EOF'
---
- name: Remove nginx
  hosts: all
  become: true
  tasks:
    - name: Stop and disable nginx
      ansible.builtin.service:
        name: nginx
        state: stopped
        enabled: false
      ignore_errors: true

    - name: Remove nginx (Arch)
      community.general.pacman:
        name: nginx
        state: absent
      when: ansible_os_family == "Archlinux"

    - name: Remove nginx (RHEL)
      ansible.builtin.dnf:
        name: nginx
        state: absent
      when: ansible_os_family == "RedHat"

    - name: Remove firewalld http rule (RHEL)
      ansible.posix.firewalld:
        service: http
        permanent: true
        state: disabled
        immediate: true
      when: ansible_os_family == "RedHat"
EOF
```
#6. Rhel 10 Hardening

See the code rhel_hardening.yml

Command
```
ansible-playbook --ask-become-pass rhel_hardening.yml
```
#7. User Management

See the code user_management.yml

Command
```
ansible-playbook --ask-become-pass user_management.yml
```

Note: sysadmin account uses SSH key authentication only, no password login.

Test access with:
```
ssh -i ~/.ssh/id_ed25519 -p 7001 sysadmin@rhel
ssh -i ~/.ssh/id_ed25519 -p 7001 sysadmin@arch
```
#8. Configuration management via templates

See the code config_management.yml and templates/sshd_config.j2

Note: ssh_port is defined per host in inventory.ini. Change to a non-standard port to reduce brute force exposure. Port 22 is a security risk in production.

Command
```
ansible-playbook --ask-become-pass config_management.yml
```

#9. Patch Management with Reboot Handling

See the code patch_management.yml

Command
```
ansible-playbook --ask-become-pass patch_management.yml
```

Note: Servers will automatically reboot if a kernel update is detected. Reboot timeout is set to 300 seconds.

#10. Compliance Reporting

See the code compliance_report.yml

Reports are saved to reports/ directory on the control node.

Command
```
ansible-playbook --ask-become-pass compliance_report.yml
```

#11. Application Deployment with Rollback

See the code app_deployment.yml

Uses symlink pattern: /opt/releases/<version> with /opt/current pointing to active release.

Rollback automatically repoints symlink to previous release on failure.

Keeps last 3 releases, cleans up older ones.

Commands
```
ansible-playbook --ask-become-pass app_deployment.yml
ansible-playbook --ask-become-pass app_deployment.yml -e "app_version=2.0.0"
```

Testing rollback (simulate failure):
```
ansible-playbook --ask-become-pass app_deployment_test.yml
```

Note: app_deployment_test.yml uses /bin/false to simulate a failed deployment and trigger the rescue/rollback block.
