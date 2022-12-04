# Networking

## Kubernetes network model
- Container to container communication -> Inter process communication (IPC)
- Pod to pod communication. Using NAT.
- Pod to service communication. Services expose a single stable DNS name.
- Node to node communication.

The specification for the Kubernetes networking model is called Container Network Interface (CNI). Network plugins that implement the CNI specification are widely available and can be configured in a Kubernetes cluster by the administrator.

## Services

They provide load balancing!

Types:
- ClusterIP (only reachable within the cluster)
- NodePort 
- LoadBalancer

```bash
k run <name> --image=<img> --restart=Never \
    --port=8080
k create service clusterip <name> --tcp=80:8080
```

The *--expose* flag for *run* command automatically creates a service for the pod.

For deployments:

```bash
k create deployment <name> --image=<img> \
    --replicas=5
k expose deployment <name> --port=80 --target-port=8080
```

## Port mapping 

### NodePort

```bash
k get nodes -o \
    jsonpath='{ $.items[*].status.addresses[?(@.type=="InternalIP")].address }'
```

## Ingress

An Ingress cannot work without an Ingress controller. The Ingress controller evaluates the collection of rules defined by an Ingress that determine traffic routing.

```bash
k create ingress <name> \
    --rule="star-alliance.com/corellian/api=corellian:8080"
```

## CoreDNS

Kubernetes runs a DNS server implementation called CoreDNS that maps the name of the Service to its IP address.

```bash
kubectl get configmaps coredns -n kube-system -o yaml
```

We could customize this CoreDNS creating the coredns-custom ConfigMap.

Svc name for other namespaces follow the syntax <svc-name>.<namespace>

## CNI Plugin

- https://chrislovecnm.com/kubernetes/cni/choosing-a-cni-provider/

