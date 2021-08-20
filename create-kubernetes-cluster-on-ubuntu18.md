# Create Kubernetes cluster on Ubuntu 18.04

[![hackmd-github-sync-badge](https://hackmd.io/QfFi_RYGQsWw54iF0KcLag/badge)](https://hackmd.io/QfFi_RYGQsWw54iF0KcLag)


This guide works for *ubuntu 18.04, kubernetes 1.20, docker 0.19.3* on *Aug. 2021*

Table of contents:
- [Pre-requirements](#Pre-requirements)
- [Creating a cluster](#Creating-a-cluster)
- [Schedule GPU (NVIDIA)](#Schedule-GPU-(NVIDIA))
- [Removing a cluster](#Removing-a-cluster)

## Pre-requirements

1. install docker 
2. install nvidia-docker 2.0 to use GPU
3. (and other perhaps) https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/

## Creating a cluster

https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/

https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/

### Disable swap
***(on all nodes)***
```
sudo swapoff -a
```

to disable swap on reboot:
```
sudo vim /etc/fstab
```
, comment out lines with `swap`

### Install kubelet, kubeadm and  kubectl
***(on all nodes)***
```
sudo apt-get update && sudo apt-get install -y apt-transport-https curl`
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
cat <<EOF | sudo tee /etc/apt/sources.list.d/kubernetes.list
deb https://apt.kubernetes.io/ kubernetes-xenial main
EOF
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
```

### Init control plane
***(on master node)***
```
sudo kubeadm init --pod-network-cidr=10.244.0.0/16
```
(`--pod-network-cidr=10.244.0.0/16` is for flannel)

```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

installing a Pod network add-on, we use flannel in this case
```
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```

check flannel running 
```
kubectl get pods --all-namespaces
```

(optional) control plane node isolation; allow k8s to schedule master node
```
kubectl taint nodes --all node-role.kubernetes.io/master-
```

### Join nodes
***(on worker nodes)***

joining nodes using token from kubeadm init
```
kubeadm join --token <token> <control-plane-host>:<control-plane-port> --discovery-token-ca-cert-hash sha256:<hash>
```

(if forgot token) re-generate via
```
kubeadm token create --print-join-command
```

check nodes
```
kubectl get nodes -o wide
```


## Schedule GPU (NVIDIA)

https://kubernetes.io/docs/tasks/manage-gpus/scheduling-gpus/

https://github.com/NVIDIA/k8s-device-plugin

### For nodes with GPU, change default runtime to `nvidia`
***(on worker nodes with GPU)***
```
sudo vim /etc/docker/daemon.json
```
modified content:
```
{
    "default-runtime": "nvidia",
    "runtimes": {
        "nvidia": {
            "path": "nvidia-container-runtime",
            "runtimeArgs": []
        }
    }
}
```
restart docker
```
sudo systemctl daemon-reload && sudo systemctl restart docker
```

### Deploy k8s-device-plugin via helm

***(on any node)***

install helm
```
curl https://baltocdn.com/helm/signing.asc | sudo apt-key add -
sudo apt-get install apt-transport-https --yes
echo "deb https://baltocdn.com/helm/stable/debian/ all main" | sudo tee /etc/apt/sources.list.d/helm-stable-debian.list
sudo apt-get update
sudo apt-get install helm
```

install k8s-device-plugin
```
helm repo add nvdp https://nvidia.github.io/k8s-device-plugin
helm repo update
helm install \
    --version=0.8.2 \
    --generate-name \
    nvdp/nvidia-device-plugin
```

check GPU found
```
kubectl describe node | grep nvidia
```

### Test

create `cuda-vector-add.yaml`, contents:
```
apiVersion: v1
kind: Pod
metadata:
  name: cuda-vector-add
spec:
  restartPolicy: OnFailure
  containers:
    - name: cuda-vector-add
      # https://github.com/kubernetes/kubernetes/blob/v1.7.11/test/images/nvidia-cuda/Dockerfile
      image: "k8s.gcr.io/cuda-vector-add:v0.1"
      resources:
        limits:
          nvidia.com/gpu: 1 # requesting 1 GPU
```

run
```
kubectl apply -f cuda-vector-add.yaml
```

verify result
```
kubectl get pod
```


## Removing a cluster

https://kubernetes.io/docs/tasks/administer-cluster/safely-drain-node/

### Delete worker nodes

delete all pods on the nodes
```
kubectl get pod
kubectl delete pod [pod-name]
```


drain the node
```
kubectl get nodes
kubectl drain <node name> --ignore-daemonsets
kubectl drain <node name> --delete-local-data --force --ignore-daemonsets
```

reset nodes

***(on nodes to remove)***
```
sudo kubeadm reset
sudo -i
rm -rf /etc/cni/net.d
iptables -F && iptables -t nat -F && iptables -t mangle -F && iptables -X
rm -rf $HOME/.kube/config
ip link delete cni0 
ip link delete flannel.1 
```

delete nodes
```
kubectl delete node <node name>
```

### Delete master node
***(on master node)***

same as reset nodes above
