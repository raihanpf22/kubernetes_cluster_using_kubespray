
# Kubernetes Cluster Kubespray

Kubernetes cluster installation using kubespray
In principle, the steps in this guide can be divided into the following two main procedures which are required in order to set up a new Kubernetes cluster.

Create the infrastructure
Deploy Kubernetes
Before delving into the actual steps, clone Kubespray onto your own computer, for example by using the git command-line tool. If you do not already have git installed, you can use the command below to install git on Ubuntu or other Debian-based operating systems or check the git install guide for other OS options.


## Installation

Install my-project with npm

```bash
  sudo apt install git-all
```
Then download the Kubespray package and change to the new directory.
```bash
git clone https://github.com/kubernetes-sigs/kubespray.git
cd kubespray
```
You’ll also need to install Ansible and other dependencies. Luckily, Kubespray provides a handy list of the requirements which can be used to install all prerequisites with a single command. However, for this to work, you’ll first need to have Python’s package installer, pip, available.
```bash
sudo apt install python3-pip
sudo pip3 install -r requirements.txt
```
The very first step when using Kubespray is to define the inventory - a core concept of Ansible, which is a list of servers, with hostnames or IP address, and the role they have on your setup.

# master node:

before install, better connect the master node to all node:

```bash
ssh-keygen -t rsa
```

then ssh node worker:

```bash
ssh-copy-id <IP-WORKER-NODE>
```

open sudoers on all node:

```bash
sudo vi /etc/sudoers
```
edit line allow members group on all node:
```bash
# Allow members of group sudo to execute any command
%sudo   ALL=(ALL:ALL) ALL

kubernetes1-raihan ALL=(ALL) NOPASSWD:ALL

kubernetes2-raihan ALL=(ALL) NOPASSWD:ALL

kubernetes3-raihan ALL=(ALL) NOPASSWD:ALL
```
# Or add new user to all node:

```bash
sudo adduser newuser
```
and, then run ansible using -u newuser -k -K:
```bash
ansible-playbook -i inventory/mycluster/hosts.yml --become --become-user=root cluster.yml -u newuser -k -K
```

now, 

First, copy the sample inventory definitions from the repo...

```bash
cp -rfp inventory/sample inventory/mycluster
```
... then bootstrap the inventory file details by running a command that lists all the IPs of your server (from controller to workers) (ssh all node from master node):

```bash
declare -a IPS=(167.235.49.54 159.69.35.153 167.235.73.16)
CONFIG_FILE=inventory/mycluster/hosts.yml python3 contrib/inventory_builder/inventory.py ${IPS[@]}
```
This command will name the nodes linearly as node1, node2, and node3. For a one controller node setup, edit the created file inventory/mycluster/hosts.yml file, change the name of the first node to controller, rename other nodes, and only use controller in the kube_control_plane group.

My final inventory file than looked like this:

```bash
all:
  hosts:
    controller:
      ansible_host: 167.235.49.54
      ip: 167.235.49.54
      access_ip: 167.235.49.54
    node1:
      ansible_host: 159.69.35.153
      ip: 159.69.35.153
      access_ip: 159.69.35.153
    node2:
      ansible_host: 167.235.73.16
      ip: 167.235.73.16
      access_ip: 167.235.73.16
  children:
    kube_control_plane:
      hosts:
        controller:
    kube_node:
      hosts:
        controller:
        node1:
        node2:
    etcd:
      hosts:
        controller:
        node1:
        node2:
    k8s_cluster:
      children:
        kube_control_plane:
        kube_node:
    calico_rr:
      hosts: {}
```
then run the ansible:

```bash
ansible-playbook -i inventory/mycluster/hosts.yml --become --become-user=root cluster.yml
```
after finish install the cluster, set the kubelet config:

```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

## Acknowledgements

 - [DevopsCube](https://devopscube.com/kubernetes-kubeconfig-file/#:~:text=A%20Kubeconfig%20is%20a%20YAML%20file%20with%20all,if%20you%20are%20using%20a%20managed%20Kubernetes%20cluster.)
 - [Kubernetes](https://kubernetes.io/docs/reference/kubectl/)
 - [Video Cloud Learn Hub](https://www.youtube.com/watch?v=9pLh2Tt1blc)

