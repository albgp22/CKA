# Logging

Options:

- Use a node-level logging agent
- Sidecar container for handling application logs
- Pushing logsto a logging backend from the application logic

```bash
minikube addons enable metrics-server
kubectl top nodes
kubectl top pod <name>
```

```bash
kubectl get events
```

```bash
kubectl logs incorrect-cmd-pod --previous
```

```bash
kubectl cluster-info
kubectl cluster-info dump
```

```bash
kubectl logs kube-apiserver-minikube -n kube-system
```

```bash
kubectl describe node worker-1
```

```bash
systemctl status kubelet
journalctl -u kubelet.service
systemctl restart kubelet
openssl x509 -in /var/lib/kubelet/pki/kubelet.crt -text
kubectl describe pod kube-proxy-csrww -n kube-system
kubectl describe daemonset kube-proxy -n kube-system
```

