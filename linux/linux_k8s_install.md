# Install Kubernetes with Kubespray

## Clone Kubespray

```shell script
export KUBESPRAY_DIRECTORY="${HOME}/kubespray/"
echo "export KUBESPRAY_DIRECTORY=\"${KUBESPRAY_DIRECTORY}\"" \
| tee -a "${HOME}/.bashrc"


cd "${HOME}"
rm -rf kubespray
git clone --depth 1 "https://github.com/kubernetes-incubator/kubespray.git"
cd "${KUBESPRAY_DIRECTORY}"
cp -rv inventory my_inventory

rm -fv "${KUBESPRAY_DIRECTORY}my_inventory/local/inventory"
ln -s "${ANSIBLE_HOSTS}" "${KUBESPRAY_DIRECTORY}my_inventory/local/inventory"

cp -rv "${KUBESPRAY_DIRECTORY}roles" "${KUBESPRAY_DIRECTORY}playbooks/roles"
```

### Edit `all.yml`

```shell script
nano ${KUBESPRAY_DIRECTORY}my_inventory/sample/group_vars/all/all.yml
```
```yaml
ntp_enabled: true
ntp_manage_config: true
ntp_servers:
  - "time.nist.gov"
  - "time.windows.com"
```

### Get available `containerd` versions

```shell script
apt-cache policy containerd

# Candidate: 1.7.28-0ubuntu1~24.04.1
```

### Edit `main.yml`

```shell script
nano "${KUBESPRAY_DIRECTORY}playbooks/roles/kubespray-defaults/tasks/main/download.yml"
```

```yaml
docker_containerd_version: 1.7.28
```

### Edit `k8s-cluster.yml`

```shell script
nano "${KUBESPRAY_DIRECTORY}my_inventory/sample/group_vars/k8s_cluster/k8s-cluster.yml"
```
```yaml
kube_version: v1.32.4
kube_network_plugin: calico
container_manager: docker
```

### Edit `k8s-cluster.yml`

```shell script
nano ${KUBESPRAY_DIRECTORY}my_inventory/sample/group_vars/k8s_cluster/addons.yml
```
```yaml
# Helm deployment
helm_enabled: true

# Nginx ingress controller deployment
ingress_nginx_enabled: true
ingress_nginx_host_network: true
```


### Edit `verify-settings.yml`

```shell script
nano roles/kubernetes/preinstall/tasks/0040-verify-settings.yml
```
```yaml
- name: Stop if memory is too small for masters
  assert:
    that: ansible_memtotal_mb >= 1500
  ignore_errors: "{{ ignore_assert_errors }}"
  when: inventory_hostname in groups['kube-master']
- name: Stop if memory is too small for nodes
  assert:
    that: ansible_memtotal_mb >= 1024
  ignore_errors: "{{ ignore_assert_errors }}"
  when: inventory_hostname in groups['kube-node']
```

### Install python packages

```shell script
cd "${KUBESPRAY_DIRECTORY}" && \
sudo echo

sudo apt-get remove --purge python3-cryptography pyopenssl python3-openssl

sudo apt-get install \
    --yes \
    --only-upgrade \
    python3-pip

sudo rm -f /usr/lib/python3.*/EXTERNALLY-MANAGED && \
sudo pip3 install --upgrade setuptools && \
sudo pip3 install -r requirements.txt
```

### Reboot & make cleanup

```shell script
sudo rm -rf /tmp/*
sudo rm -rf /var/lib/etcd
sudo rm -rf /etc/systemd/system/etcd
sudo shutdown -r now

sudo swapoff -a
```

### (Optional) Reset Kubernetes cluster

```shell script
export ANSIBLE_HOST_KEY_CHECKING=False

cd "${KUBESPRAY_DIRECTORY}"
ansible-playbook \
    --ask-vault-pass \
    --extra-vars "@${ANSIBLE_VAULT_FILE}" \
    --become \
    --become-user=root \
    --user ${ANSIBLE_USER_NAME} \
    --flush-cache \
    --forks $(nproc) \
    --inventory "${KUBESPRAY_DIRECTORY}my_inventory/local/inventory" \
    -vvvv \
    "${KUBESPRAY_DIRECTORY}reset.yml" \
|& tee "${KUBESPRAY_DIRECTORY}reset_cluster.log"

# or without Ansible Vault:
cd "${KUBESPRAY_DIRECTORY}"
ansible-playbook \
    --ask-pass \
    --ask-become-pass \
    --become \
    --become-user=root \
    --user ${ANSIBLE_USER_NAME} \
    --flush-cache \
    --forks $(nproc) \
    --inventory "${KUBESPRAY_DIRECTORY}my_inventory/local/inventory" \
    -vvvv \
    "${KUBESPRAY_DIRECTORY}reset.yml" |& tee "${KUBESPRAY_DIRECTORY}reset_cluster.log"


<sudo password>
yes


sudo apt-get remove --purge -y containerd.io docker*
sudo rm -rf /var/lib/etcd2/* 
sudo rm -f /etc/systemd/system/etcd* 
```

### Deploy Kubespray

```shell script
cd "${KUBESPRAY_DIRECTORY}"
ansible-playbook \
    --ask-vault-pass \
    --extra-vars "@${ANSIBLE_VAULT_FILE}" \
    --become \
    --become-user=root \
    --user ${ANSIBLE_USER_NAME} \
    --flush-cache \
    --forks $(nproc) \
    --inventory "${KUBESPRAY_DIRECTORY}my_inventory/local/inventory" \
    -vvvv \
    "${KUBESPRAY_DIRECTORY}cluster.yml" |& tee "${KUBESPRAY_DIRECTORY}kubespray_cluster.log"
```

### Get rid of the error x509

```shell script
# Run by a regular user

sudo swapoff -a

cd
export KUBECONFIG="${HOME}/.kube/config"
echo "export KUBECONFIG=\"${KUBECONFIG}\"" \
| tee -a "${HOME}/.bashrc"
# nano "${HOME}/.bashrc"

sudo rm -rf "$(dirname "${KUBECONFIG}")"
mkdir -p "$(dirname "${KUBECONFIG}")"

sudo cp -rv /etc/kubernetes/admin.conf "${KUBECONFIG}"
sudo chown "$(id -u):$(id -g)" "${KUBECONFIG}"

sudo groupadd docker;
sudo usermod -aG docker "$(whoami)"
sudo newgrp docker

sudo systemctl enable kubelet.service containerd.service docker.service cri-dockerd.service
sudo systemctl restart kubelet.service containerd.service docker.service cri-dockerd.service

sudo ufw disable
sudo iptables -P INPUT ACCEPT
sudo iptables -P OUTPUT ACCEPT
sudo iptables -P FORWARD ACCEPT
sudo iptables -F

sudo kubectl get nodes
```

### Is Kubespray deployed?

```shell script
pgrep kubelet

sudo kubectl get all
sudo kubectl get nodes 

# Get all running pods

sudo kubectl get pods --all-namespaces
```

### Assign labels to nodes

```shell script
sudo kubectl label nodes \
    --overwrite \
    node01 node02 node03 \
    disktype=ssd \
    threads=4 \
    ram=4

# Get running nodes with labels

sudo kubectl get nodes --show-labels
```

# Use Kubernetes

## Access Kubernetes

#### Create the first pod

```shell script
mkdir ~/k8s-scripts
nano ~/k8s-scripts/first-pod.yaml
```
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: first-pod
spec:
  containers:
  - name: sise
    image: mhausenblas/simpleservice:0.5.0
    ports:
    - containerPort: 9876
    resources:
      limits:
        memory: "128Mi"
        cpu: "500m"
  - name: shell
    image: centos:7
    command:
      - "bin/bash"
      - "-c"
      - "sleep 10000"
```

#### Deploy the first pod

```shell script
sudo kubectl run sise --image=mhausenblas/simpleservice:0.5.0 --port=9876
sudo kubectl apply -f ~/k8s-scripts/first-pod.yaml
```
```text
pod "first-pod" created
```

#### Is the first pod deployed?

```shell script
sudo kubectl get pods
```
```text
NAME        READY     STATUS    RESTARTS   AGE
first-pod   2/2       Running   0          20m
 ```

#### Access pod CLI

```shell script
sudo kubectl exec first-pod -c sise -i -t -- bash
```
```text
root@first-pod:/usr/src/app# curl localhost:9876/info
{"host": "localhost:9876", "version": "0.5.0", "from": "127.0.0.1"}
root@first-pod:/usr/src/app#
```

#### Launch dashboard

```shell script
sudo kubectl create -f https://raw.githubusercontent.com/kubernetes/dashboard/master/src/deploy/recommended/kubernetes-dashboard.yaml
sudo kubectl proxy
```
```shell script
firefox http://localhost:8001/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy/
```

#### Create `admin-user`

```shell script
nano ~/k8s-scripts/create_admin-user.yaml
```
```yaml
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin-user
  namespace: kube-system
```
```shell script
sudo kubectl create -f ~/k8s-scripts/create_admin-user.yaml
```
```
serviceaccount "admin-user" created
```

#### Create ClusterRoleBinding for `admin-user`

```shell script
nano ~/k8s-scripts/create_admin-user_ClusterRoleBinding.yaml
```
```yaml
---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRoleBinding
metadata:
  name: admin-user
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: admin-user
  namespace: kube-system
```
```shell script
sudo kubectl create -f ~/k8s-scripts/create_admin-user_ClusterRoleBinding.yaml
```
```text
clusterrolebinding "admin-user" created
```

#### Get the bearer token

```shell script
echo $(sudo kubectl -n kube-system describe secret $(sudo kubectl -n kube-system get secret | grep admin-user | awk '{print $1}')) >> ~/k8s-scripts/token_login_admin-user.secret
nano ~/k8s-scripts/token_login_admin-user.secret
# Copy the latest token and paste it into Dashboard
```

#### Make the dashboard accessible from external network

```shell script
sudo kubectl proxy --address 0.0.0.0 --accept-hosts '.*'

# External URL:
# http://192.168.188.131:8001/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy/#!/login
# However, login attempt may be perforned from the host domain only, see:

# https://github.com/kubernetes/dashboard/issues/2540
```

# Taint master node to run containers

```shell script
kubectl taint node mymasternode node-role.kubernetes.io/control-plane:NoSchedule-
```
