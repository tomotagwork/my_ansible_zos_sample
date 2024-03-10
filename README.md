# Overview

This Repository contains sample Ansible playbooks that use Red Hat Ansible Certified Content for IBM Z.


# Requirement

## Ansible control node

- Ansible 2.15.x


## Target z/OS

- z/OS OpenSSH
- IBM Open Enterprise Python for z/OS and IBM Z Open Automation Utilities


# Setup Instruction

1. Clone sample repository

```sh
git clone https://github.com/tomotagwork/my_ansible_zos_sample.git
```

2. Install collections in project local

```sh
ansible-galaxy install -r requirements.yml
```

3. Configure connection info

Modify inventory.yml like below.

```yaml
all:
  children:
    ungrouped: {}
    zos:
      hosts:
        <hostname>:
          ansible_host: xxx.xxx.xxx.xxx
          ansible_user: IBMUSER
          ansible_port: 22
          ansible_ssh_private_key_file: "~/.ssh/ssh_private_key.pem"
```

4. Issue sample playbook

```sh
ansible-playbook sample/zos_operator.yml 
```
