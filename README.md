# Install

```
# install
pipx install --include-deps ansible

# upgrade
pipx upgrade --include-injected ansible
```


Inventory file (can be in the local directory): more details below
```
$ nano hosts
[vms]
narayani ansible-host=10.10.x.x ansible_port=xx ansible_user=some-user
kyanjin ansible_host=10.10.x.x ansible_port=xx ansible_user=some-user
```

# Ad-hoc commands

```
ansible all --key-file ~/.ssh/ansible-key -i inventory -m ping --ask-become-pass

ansible all -i inventory -m gather_facts

ansible all -i inventory -m gather_facts --limit kyanjin | grep ansible_distribution
```

## Elevated ad-hoc commands

```
ansible all -i inventory -m apt -a update_cache=yes --become --ask-become-pass

upgrade=yes/dist 
name=sl
"name=sl state=absent/latest"
```

# Ansible-playbook

```
$ nano install_sl.yml

---

- hosts: all
  become: true
  tasks:

  - name: update apt repository index
    apt:
      update_cache: yes
    when: ansible_distribution in ["Debian", "Ubuntu"]
    # combine whens with: and ansible_distribution_version == "22.04"

  - name: install sl package
    apt:
      name: sl
      state: latest
    when: ansible_distribution in ["Debian", "Ubuntu"]
```

```
ansible-playbook --ask-become-pass -i inventory install_sl.yml 
```


## Tags

```
$ nano install.yml

---

- hosts: all
  become: true
  tasks:

  - name: update packages
    tags: always
    apt:
      update_cache: yes
    when: ansible_distribution in ["Debian", "Ubuntu"]


  - name: update and install stress and s-tui
    tags: ubuntu,debian,stess,s-tui
    apt:
      name:
        - stress
        - s-tui
      state: latest
    when: ansible_distribution in ["Debian", "Ubuntu"]

```


```
ansible-playbook --tags "htop,powertop" -i inventory install.yml --ask-become-pass
```

## Copy files

```
$ touch files/file-to-copy.txt
$ nano copy.yml

---

- hosts: test
  tasks:

  - name: copy file
	​copy:
      src: file-to-copy.txt
      dest: ~/destination-name.txt
      owner: shree
      group: shree
      mode: 0644
```

```
ansible-playbook -i inventory copy.yml
```


Define host name ip address etc

nano hosts
```
[all:vars]
ansible_user='some-user'
ansible_become=yes
ansible_become_method=sudo
ansible_python_interpreter='/usr/bin/env python3'

[ubuntutest]
ubuntu-test

[dockertest]
docker-test ansible_host=10.10.xx.xx ansible_port=xxxx ansible_user=shreead
```

List inventory
```
ansible-inventory -i hosts --list
```

ubuntu-test defined in .ssh/config

## Usage
```
ansible all -i hosts -m ping
```




# Playbook yaml file

```
---
- hosts: all  # or hosts: ubuntutest
  become: true
#  become_user: shree  # not sure what this does
  tasks:
    - name: ...
      ...
```

## To run
`ansible-playbook -i hosts playbook-name.yaml --ask-become-pass --limit baremetal`

# Tasks

## Package management

#### apt update
```
    - name: apt update
      apt: update_cache=yes force_apt_get=yes cache_valid_time=3600
```

#### apt upgrade -y
```
    - name: apt upgrade
      apt: upgrade=dist force_apt_get=yes
```

#### Check if reboot is required
```
    - name: Check if a reboot is needed on all servers
      register: reboot_required_file
      stat: path=/var/run/reboot-required
```

#### Reboot if neccessary
```
    - name: Reboot the box if kernel updated
      reboot:
        msg: "Reboot necessary for kernel updates"
        connect_timeout: 5
        reboot_timeout: 300
        pre_reboot_delay: 0
        post_reboot_delay: 30
        test_command: uptime
      when: reboot_required_file.stat.exists
```

#### Install packages
```
    - name: Install packages
      apt:
        name:
          - htop
          - powertop
        state: latest
        update_cache: true

```

#### Remove packages
```
  - name: remove packages
    apt:
      name:
      - stress
      - s-tui
      state: absent
```

## Git

#### Clone git repo
```
    - name: Clone a git repository
      git:
        repo: https://github.com/user/repo.git
        dest: /path/to/repo/
        clone: yes
        update: yes
```


## File operations

#### Create file
```
  - name: Create file
    file:
      path: /path/to/file.txt
      owner: some-user
      group: some-user
      access_time: preserve
      modification_time: preserve
      state: touch
```

#### Add text to file
```
  - name: add a block of text
    blockinfile:
      path: /path/to/file.txt
      block: |
        First line
        Second line
        Third line

```

#### Copy file
```
  - name: copy file-to-copy file
    copy:
      src: file-to-copy.txt
      dest: /path/to/file.txt
      owner: some-user
      group: some-user
      mode: 0644

```
#### Delete file
```
    - name: Remove a file
      file:
        path:
          - /path/to/file.txt
        state: absent
```


## Shell commands

Run shell command
```
    - name: shell command to stow files
      ansible.builtin.shell:
        cmd: |
          stow -d /home/shree/dotfiles/ -t /home/shree -S bash
          stow -d /home/shree/dotfiles/ -t /home/shree/.ssh -S ssh
          rm /home/shree/.alias-laptop
```


## Misc

#### Remove MOTD ad
```
  - name: Remove MOTD ad
    lineinfile:
      path: /etc/default/motd-news
      regexp: ^ENABLED
      line: ENABLED=0

  - name: Remove execution permission from /etc/update-motd.d/10-help-text
    file:
      path: /etc/update-motd.d/10-help-text
      mode: '0644'

```


