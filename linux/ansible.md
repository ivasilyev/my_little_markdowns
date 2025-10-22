# Setup Ansible

## Prepare environment

```
# Before run edit ssh_config and /etc/hosts

# Run under regular user!

unset HISTFILE
sudo echo
cd

echo Export variables

export TOOL_NAME="ansible"
export UN="user"
export UPW="qwerty"

export LOCAL_ENV="${HOME}/.bashrc"
export SYS_ENV="/etc/environment"

# Edit the bash profiles if required: 
# nano "${LOCAL_ENV}"
# sudo nano "${SYS_ENV}"

echo Create system Ansible config variable
export ANSIBLE_CONFIG="/etc/ansible/ansible.cfg"
echo "export ANSIBLE_CONFIG=\"${ANSIBLE_CONFIG}\"" \
| tee -a "${LOCAL_ENV}" \
| sudo tee -a "${SYS_ENV}"

echo Create hosts variable
export ANSIBLE_HOSTS="/etc/ansible/hosts.yml"
echo "export ANSIBLE_HOSTS=\"${ANSIBLE_HOSTS}\"" \
| tee -a "${LOCAL_ENV}" \
| sudo tee -a "${SYS_ENV}"

echo Create Ansible directory variable
export ANSIBLE_DIRECTORY="${HOME}/.ansible"
echo "export ANSIBLE_DIRECTORY=\"${ANSIBLE_DIRECTORY}\"" \
| tee -a "${LOCAL_ENV}" \
| sudo tee -a "${SYS_ENV}"

echo Create playbooks directory variable
export ANSIBLE_PLAYBOOK_DIRECTORY="${ANSIBLE_DIRECTORY}/playbooks/"
echo "export ANSIBLE_PLAYBOOK_DIRECTORY=\"${ANSIBLE_PLAYBOOK_DIRECTORY}\"" \
| tee -a "${LOCAL_ENV}" \
| sudo tee -a "${SYS_ENV}"

echo Create vault variable
export ANSIBLE_VAULT_FILE="${ANSIBLE_DIRECTORY}/vault.yml"
echo "export ANSIBLE_VAULT_FILE=\"${ANSIBLE_VAULT_FILE}\"" \
| tee -a "${LOCAL_ENV}" \
| sudo tee -a "${SYS_ENV}"

echo Create remote user name variable
export ANSIBLE_USER_NAME="${UN}"
echo "export ANSIBLE_USER_NAME=\"${ANSIBLE_USER_NAME}\"" \
| tee -a "${LOCAL_ENV}" \
| sudo tee -a "${SYS_ENV}"
```

# Install Ansible

## (Optional) Remove Ansible

```
echo Remove Ansible && \
sudo apt-get remove \
    --yes \
    --purge \
    ansible && \
sudo pip uninstall -y ansible && \
sudo apt-get autoremove -y && \
sudo dpkg --configure -a
```

## Install software

```
sudo apt-get update -y; sudo apt-get upgrade -y; sudo apt-get clean; sudo apt-get autoclean; sudo apt-get autoremove -y

# Fix ModuleNotFoundError: No module named 'debian'
sudo apt-get --reinstall install python3-debian
sudo dpkg --configure -a
sudo apt \
    --fix-broken \
    install \
    --yes

echo Install Ansible && \
sudo apt-get update \
    --yes && \
sudo apt-get install \
    --yes \
    python3-dev \
    python3-pip \
    python3-debian \
    software-properties-common \
    sshpass && \
sudo apt-add-repository \
    --yes \
    --update \
    ppa:ansible/ansible && \
sudo apt-get install \
    --yes \
    ansible && \
ansible-config \
    init \
    --disabled \
    --format ini \
    --type all \
| sudo tee "${ANSIBLE_CONFIG}" && \
echo Check Ansible version && \
ansible --version \
| head -n 1

echo Setting up Ansible bash completion support && \
sudo apt install -y python3-argcomplete && \
sudo activate-global-python-argcomplete
```

## Tweak Ansible performance

```
sudo nano "${ANSIBLE_CONFIG}"
```
```
# Change carefully, line by line for each corresponding section
# strategy=free
strategy=linear
forks=99
inventory=/etc/ansible/hosts.yml,
remote_user=ansible
timeout=180
```

## Create inventory

```
mkdir \
    --parents \
    --mode 755 \
    --verbose \
    "${ANSIBLE_DIRECTORY}" \
    "${ANSIBLE_PLAYBOOK_DIRECTORY}"

# See 'hosts_*.yml'
sudo nano "${ANSIBLE_HOSTS}"

cd "${ANSIBLE_DIRECTORY}"

# EDITOR=nano ansible-vault create "${ANSIBLE_VAULT_FILE}"
EDITOR=nano ansible-vault edit "${ANSIBLE_VAULT_FILE}"
# Add during execution: --ask-vault-pass --extra-vars "@${ANSIBLE_VAULT_FILE}"
```
```
---
ansible_ssh_user: "ansible"
ansible_ssh_pass: ""
ansible_password: ""
ansible_become_pass: ""
```
