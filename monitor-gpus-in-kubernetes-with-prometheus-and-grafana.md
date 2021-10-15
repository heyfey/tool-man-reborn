# Monitor GPUs in Kubernetes with Prometheus and Grafana 

Just check out the links :)

https://docs.nvidia.com/datacenter/cloud-native/kubernetes/dcgme2e.html

[Prometheus and Grafana](https://docs.nvidia.com/datacenter/cloud-native/kubernetes/dcgme2e.html#gpu-telemetry)

[heyfey/nvidia_smi_exporter](https://github.com/heyfey/nvidia_smi_exporter)

## View Grafana locally

### In cluster

list all service
```
kubectl get svc -A
```

expose grafana with port forwarding

```
kubectl port-forward svc/kube-prometheus-stack-1625052244-grafana -n prometheus 32322:80
```

output:

```
Forwarding from 127.0.0.1:32322 -> 3000
Forwarding from [::1]:32322 -> 3000
```

### On the local

Start a local terminal (with something likes **MobaXterm**)

```
ssh <username>@<server-ip> -N -L 32322:localhost:32322
```

In browser:
http://localhost:32322/
