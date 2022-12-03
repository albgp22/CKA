# Managing a Kubernetes Cluster

The low-level command-line tool for performing cluster bootstrapping operations is called kubeadm. It is not meant for provisioning the underlying infrastructure. That’s the purpose of infrastructure automation tools like Ansible and Terraform.

While not explicitly stated in the CKA frequently asked questions (FAQ) page, you can assume that the kubeadm executable has been preinstalled for you. The following sections describe the processes for creating and managing a Kubernetes cluster on a high level and will use kubeadm heavily.

## Installing a cluster

Process:

1. (control plane node)
    - SSH into machine
    - kubeadm init
    - copy kubeconfig for non-root users
    - install CNI
    - check node status
2. (worker node)
    - SSH into machine
    - kubeadm join
    - SCP kubeconfig
    - check node status
    - exit
3. (worker node)
    - repeat

```bash
ssh kube-control-plane
sudo kubeadm init --pod-network-cidr 172.18.0.0/16 \
    --apiserver-advertise-address 10.8.8.10
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```


The CKA exam will most likely ask you to install a specific add-on. Most of the installation instructions live on external web pages, not permitted to be used during the exam. Make sure that you search for the relevant instructions in the official Kubernetes documentation. For example, you can find the installation instructions for Weave Net here. The following command installs the Weave Net objects:

```bash
$ kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version= \
    $(kubectl version | base64 | tr -d '\n')"
```

### Joining the Worker Nodes

```bash
kubeadm join 10.8.8.10:6443 --token fi8io0.dtkzsy9kws56dmsp \
    --discovery-token-ca-cert-hash \
    sha256:cc89ea1f82d5ec460e21b69476e0c052d691d0c52cce83fbd7e403559c1ebdac
```

## Managing a Highly Available Cluster

High-availability (HA) clusters help with scalability and redundancy. Given the complexity of standing up an HA cluster, it’s unlikely that you’ll be asked to perform the steps during the exam. 

The external etcd node topology separates etcd from the control plane node by running it on a dedicated machine.

## Upgrading Cluster Version

1. (control plane node)
    - install new kubeadm version
    - kubeadm upgrade plan
    - kubeadm upgrade apply
    - drain workload on node
    - upgrade kubelet and kubectl
    - restart kubelet process
    - allow new workload
2. (control plane node)
    - same but using kubeadm upgrade node instead of kubeadm upgrade apply
3. (worker)
    - similar with kubeadm upgrade node

```bash
sudo apt-cache madison kubeadm
sudo apt-mark unhold kubeadm && sudo apt-get update && sudo apt-get install \
    -y kubeadm=1.19.0-00 && sudo apt-mark hold kubeadm
sudo apt-get update && sudo apt-get install -y --allow-change-held-packages \
    kubeadm=1.19.0-00
sudo kubeadm upgrade plan
sudo kubeadm upgrade apply v1.19.0
kubectl drain kube-control-plane --ignore-daemonsets
sudo apt-mark unhold kubelet kubectl && sudo apt-get update && sudo \
    apt-get install -y kubelet=1.19.0-00 kubectl=1.19.0-00 && sudo apt-mark \
    hold kubelet kubectl
sudo systemctl daemon-reload
sudo systemctl restart kubelet
kubectl uncordon kube-control-plane
```

```bash
sudo apt-mark unhold kubeadm && sudo apt-get update && sudo apt-get install \
    -y kubeadm=1.19.0-00 && sudo apt-mark hold kubeadm
kubeadm version
sudo kubeadm upgrade node
kubectl drain kube-worker-1 --ignore-daemonsets
sudo apt-mark unhold kubelet kubectl && sudo apt-get update && sudo apt-get \
    install -y kubelet=1.19.0-00 kubectl=1.19.0-00 && sudo apt-mark hold kubelet \
    kubectl
sudo systemctl daemon-reload
sudo systemctl restart kubelet
kubectl uncordon kube-worker-1
kubectl get nodes
```

# B&R etcd

Using etcdctl tool.

## Backing up:

```bash
etcdctl version
kubectl get pods -n kube-system
kubectl describe pod etcd-kube-control-plane -n kube-system
kubectl describe pod etcd-kube-control-plane -n kube-system
sudo ETCDCTL_API=3 etcdctl --cacert=/etc/kubernetes/pki/etcd/ca.crt \
    --cert=/etc/kubernetes/pki/etcd/server.crt \
    --key=/etc/kubernetes/pki/etcd/server.key \
    snapshot save /opt/etcd-backup.db
```

## Restoring

```bash
sudo ETCDCTL_API=3 etcdctl --data-dir=/var/lib/from-backup snapshot restore \
    /opt/etcd-backup.db
sudo ls /var/lib/from-backup
cd /etc/kubernetes/manifests/
sudo vim etcd.yaml
kubectl get pod etcd-kube-control-plane -n kube-system
```
