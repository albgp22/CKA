# Role-Based Access Control (RBAC)

RBAC helps with implementnting a variety of uses cases:

- Establishing a system for users with different roles to access a set of Kubernetes resources
- Controlling processes running in a Pod and the operations they can perform via the Kubernetes API
- Limiting the visibility of certain resources per namespace

Subject/Resource/Verb

## Creating a subject

It can be a user account, a service account or a group as a subject. Meant to be managed by the administrator of a Kubernetes cluster. Users and groups are not stored in etcd.

Calls to an API server with a user need to be authenticated. Ways:
- X.509 client certificate
- Basic authentication
- Bearer token

### Client certificate

For creating a user that uses a client certificate to authenticate:

```bash
$ mkdir cert && cd cert
$ openssl genrsa -out johndoe.key 2048
$ openssl req -new -key johndoe.key -out johndoe.csr -subj \
    "/CN=johndoe/O=cka-study-guide"
$ openssl x509 -req -in johndoe.csr -CA /.minikube/ca.crt -CAkey \
    /.minikube/ca.key -CAcreateserial -out johndoe.crt -days 364
```

> CSR = certificate sign request

```bash
$ kubectl config set-credentials johndoe \
    --client-certificate=johndoe.crt --client-key=johndoe.key
$ kubectl config set-context johndoe-context --cluster=minikube \
    --user=johndoe
$ kubectl config use-context johndoe-context
$ kubectl config current-context
```

### Service Account

```bash
$ kubectl create serviceaccount build-bot
```

K8s creates the corresponding secret for the SA

Assigning SA to a Pod:

```bash
$ kubectl run build-observer --image=alpine --restart=Never \
--serviceaccount=build-bot
```

or using the spec.serviceAccountName field in a Pod/Deployment/Job/CronJob.

## RBAC API Primitives

- *Role*

```
The Role API primitive declares the API resources and their operations this rule should operate on. For example, you may want to say “allow listing and deleting of Pods,” or you may express “allow watching the logs of Pods,” or even both with the same Role. Any operation that is not spelled out explicitly is disallowed as soon as it is bound to the subject.
```

- *RoleBinding*

```
The RoleBinding API primitive binds the Role object to the subject(s). It is the glue for making the rules active. For example, you may want to say “bind the Role that permits updating Services to the user John Doe.”
```

### Default roles

Cluster-admin,admin,edit,view

### Creating roles and roleBindings

```bash
$ kubectl create role read-only --verb=list,get,watch \
    --resource=pods,deployments,services
```

```bash
$ kubectl create rolebinding read-only-binding --role=read-only --user=johndoe
```

### List my permissions

```bash
$ kubectl auth can-i --list --as johndoe
$ kubectl auth can-i list pods --as johndoe
```

### Aggregating RBAC Rules

ClusterRoles can specify an aggregationRule.

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: pods-services-aggregation-rules
  namespace: rbac-example
aggregationRule:
  clusterRoleSelectors:
  - matchLabels:
      rbac-pod-list: "true"
- matchLabels:
      rbac-service-delete: "true"
rules: [] # This should be empty!
```