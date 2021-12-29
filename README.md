# k3s-ansible-ha
Fork from https://github.com/k3s-io/k3s-ansible

# Build a Kubernetes cluster using k3s via Ansible

Author: <https://github.com/itwars>

## K3s Ansible Playbook

Build a Kubernetes cluster using Ansible with k3s. The goal is easily install a Kubernetes cluster on machines running:

- [X] Debian
- [X] Ubuntu
- [X] CentOS

on processor architecture:

- [X] x64
- [X] arm64
- [X] armhf

## System requirements

Deployment environment must have Ansible 2.4.0+
Master and nodes must have same sudo password SSH access or passwordless SSH access

## Usage

##Install keepalived before haproxy

First create a new directory based on the `sample` directory within the `inventory` directory:

```bash
cp -R inventory/sample inventory/my-cluster
```

Second, edit `inventory/my-cluster/hosts.ini` to match the system information gathered above. For example:

```bash
[master]
192.16.35.12

[node]
192.16.35.[10:11]

[k3s_cluster:children]
master
node
```

If needed, you can also edit `inventory/my-cluster/group_vars/all.yml` to match your environment.

Start provisioning of the cluster using the following command:

```bash
#Dry-run with ssh password and sudo password
ansible-playbook site.yml -i inventory/my-cluster/hosts.ini -CD -b -k -K
#Apply changes (if any) with ssh password and sudo password
ansible-playbook site.yml -i inventory/my-cluster/hosts.ini -D -b -k -K
```

## Kubeconfig

To get access to your **Kubernetes** cluster just

```bash
scp rke@master_ip:~/.kube/config ~/.kube/config
```

## Upgrade 1 mater 1.21.5 with Embedded SQLite to 1.22.2 3 masters with Embedded ETCD DB

Upgrade the 1.21.5 to 1.22.2

```bash
ansible <HOST-IP> -m asible.builtin.file -a "dest=/usr/local/bin/k3s state=absent"
# Leave the 1 master in the hosts.ini
ansible ansible-playbook -i inventory/my-cluster/hosts.ini  site.yml -D -b -k -K

#Add the token and --cluster-init to the k3s.service and restart
cat /var/lib/rancher/server/token
k3s server --token xxx::server:xx --no-deploy traefik --docker --cluster-init

# Add the other masters to the hosts.ini
ansible ansible-playbook -i inventory/my-cluster/hosts.ini  site.yml -D -b -k -K

#Swap agent service: Because the worker1 is v1.21.5 and the new cluster can't get the mertics about it, it's using the k3s-agent.service,login the worker1
systemctl stop k3s-agent
systemctl disable k3s-agent
systemctl start k3s-node
```

