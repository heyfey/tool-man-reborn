# Create VMs for testing

### download and create image
```
curl -O http://releases.ubuntu.com/20.04/ubuntu-20.04.2-desktop-amd64.iso

qemu-img create -f qcow2 image_file.qcow2 20G
```

### create master node
```
virt-install \
 --name k8s-master \
 --memory 4096 \
 --vcpus 2 \
 --disk size=30 \
 --cdrom ubuntu-20.04.2-desktop-amd64.iso \
 --os-variant ubuntu20.04
```

### create worker node(s)
```
virt-install \
 --name k8s-node-01 \
 --memory 4096 \
 --vcpus 2 \
 --disk size=30 \
 --cdrom ubuntu-20.04.2-desktop-amd64.iso \
 --os-variant ubuntu20.04
```