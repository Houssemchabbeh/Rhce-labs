## Lab Objective
Create an Ansible playbook that provisions users across different environments (dev/test/prod) using vault-encrypted passwords based on job roles.

## Scenario
A company needs to automate user creation across their infrastructure based on their job role using pre-created vault file with passwords and user data. Users in different environments require different access levels, and passwords must be securely stored using Ansible Vault.

## Prerequisites
- Ansible installed on control node
- Access to managed nodes (dev, test, prod hosts)
- Basic understanding of Ansible Vault
- Knowledge of user module and group management

  ---
## Directory structure
/home/greg/ansible/
├── 📄 users.yml              # Main playbook
├── 📄 vault.yml              # Pre-created vault file with passwords
├── 📄 secret.txt              # Vault password file
├── 📄 user_list.yml           # Downloaded user data
├── 📄 inventory.yml            # Inventory

  ---

### 📄secret.txt
VAULT_PASSW@RD

### 📄user_list.yml
users:
- name: adam
  uid: 3000
  job: developer
- name: ali
  uid: 3001
  job: manager

ansible-vault create vault.yml --vault password-file=secret.txt 
### 📄vault.yml
pass_developer: Imadeveloper
pass_manager: Imamanager

### 📄 users.yml
- name: create users with specific password and configuration
  hosts: all
  vars_files:
    - user_list.yml
    - vault.yml
  tasks:
    - name: create opsdev group
      ansible.builtin.group:
        name: opsdev
        state: present
    - name: Add user in opsdev group
      ansible.builtin.user:
        name: "{{ item.name }}"
        uid: "{{ item.uid }}"
        groups: opsdev
        password: "{{ pass_developer | password_hash('sha512') }}"
      loop: "{{ users }}"
      when: item.job == "developer" and inentory_hostname in groups.dev

    - name: create opsmgr group
      ansible.builtin.group:
        name: opsmgr
        state: present
    - name: Add user in opsmgr group
      ansible.builtin.user:
        name: "{{ item.name }}"
        uid: "{{ item.uid }}"
        groups: opsmgr
        password: "{{ pass_manager | password_hash('sha512') }}"
      loop: "{{ users }}"
      when: item.job == "manager" and inentory_hostname in groups.prod
  ---

ansible-playbook users.yml --vault-password-file=secret.txt

## Expected Outcome
After running the playbook:
- ✅ adam created on dev server with uid 3000
- ✅ ali created on prod server with uid 3001  
- ✅ Both users have SHA512 encrypted passwords

# Check if users exist and verify group
ansible dev -m command -a "id adam"
ansible dev -m command -a "groups adam"
ansible prod -m command -a "id ali"
ansible dev -m command -a "groups adam"

