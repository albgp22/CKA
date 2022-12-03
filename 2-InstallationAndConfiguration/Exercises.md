1. Create SA named api-access in a new namespace *apps*.

```bash
k create sa api-access -n apps
```

2. Create CR and CRB

```bash
k create clusterrole api-clusterrole -n apps --verb=watch,list,get --resource=pod

k create clusterrolebinding --clusterrole=api-clusterrole --serviceaccount=apps:api-access api-clusterrolebinding
```

3. Create POD

```bash
k run -n apps  --image=nginx:1.21.1 --port=80 --overrides='{ "spec": { "serviceAccount": "api-access" }  }' operator

k run -n disposable  --image=nginx:1.21.1 --port=80  rm
```

4. 

```bash
k config view --minify -o \
    jsonpath='{.clusters[0].cluster.server}'
kubectl get secret $(kubectl get serviceaccount api-access -n apps \
    -o jsonpath='{.secrets[0].name}') -o jsonpath='{.data.token}' -n apps | 
    base64 --decode
k exec -n apps operator -it -- /bin/bash

curl -k https://${KUBERNETES_SERVICE_HOST}:443/api/v1/pods -H "Authorization: Bearer $(cat /var/run/secrets/kubernetes.io/serviceaccount/token)" ->
<SUCCESS>

k exec -n disposable rm -it -- /bin/bash

curl -k https://$KUBERNETES_SERVICE_HOST:443/api/v1/pods -H "Authorization: Bearer $(cat /var/run/secrets/kubernetes.io/serviceaccount/token)" ->
{
  "kind": "Status",
  "apiVersion": "v1",
  "metadata": {},
  "status": "Failure",
  "message": "pods is forbidden: User \"system:serviceaccount:disposable:default\" cannot list resource \"pods\" in API group \"\" at the cluster scope",
  "reason": "Forbidden",
  "details": {
    "kind": "pods"
  },
  "code": 403
}

```