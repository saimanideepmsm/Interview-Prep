# Kubernetes Namespaces

## 1. What is a Namespace?

A **Namespace** in Kubernetes is a logical way to divide and organize resources inside the same Kubernetes cluster.

Think of a Namespace as a **separate workspace/environment inside a Kubernetes cluster**.

For example:

```text
Kubernetes Cluster
│
├── dev
│   ├── nginx
│   ├── backend
│   └── database
│
├── test
│   ├── nginx
│   └── backend
│
└── prod
    ├── nginx
    ├── backend
    └── database
```

The same cluster can contain multiple environments using different Namespaces.

---

# 2. Why Do We Need Namespaces?

Namespaces are mainly useful for:

### 2.1 Naming and Organization

Resources can have the same name if they are in different Namespaces.

For example:

```text
dev namespace
└── pod: nginx

prod namespace
└── pod: nginx
```

Both Pods can be called `nginx` because they belong to different Namespaces.

Without Namespaces:

```text
nginx
nginx    ❌ Duplicate name
```

With Namespaces:

```text
dev/nginx
prod/nginx    ✅
```

---

## 2.2 Resource Quotas

We can control how much **CPU and memory** a Namespace can consume.

For example:

```text
dev namespace
CPU    → 4 cores maximum
Memory → 8 Gi maximum

prod namespace
CPU    → 16 cores maximum
Memory → 32 Gi maximum
```

This prevents one environment/team from consuming all cluster resources.

Example:

```yaml
apiVersion: v1
kind: ResourceQuota

metadata:
  name: dev-quota
  namespace: dev

spec:
  hard:
    requests.cpu: "2"
    requests.memory: 4Gi
    limits.cpu: "4"
    limits.memory: 8Gi
```

This ResourceQuota applies only to the `dev` Namespace.

---

## 2.3 Access Control

Namespaces can be used together with **RBAC (Role-Based Access Control)** to restrict access.

For example:

```text
Dev Team
   │
   └── Can access dev namespace

Admin
   │
   ├── Can access dev
   ├── Can access test
   └── Can access prod
```

Example:

```text
developer
    ↓
dev namespace ✅
prod namespace ❌

admin
    ↓
dev namespace ✅
test namespace ✅
prod namespace ✅
```

Important:

> A Namespace itself does not provide security/access control. RBAC is used to control who can access resources within a Namespace.

---

# 3. Default Kubernetes Namespaces

When Kubernetes is initially installed, several Namespaces are normally available.

Check them using:

```bash
kubectl get namespaces
```

or:

```bash
kubectl get ns
```

Typical output:

```text
NAME              STATUS   AGE
default           Active   ...
kube-node-lease   Active   ...
kube-public       Active   ...
kube-system       Active   ...
```

---

# 4. `default` Namespace

The `default` Namespace is used when you don't explicitly specify a Namespace.

For example:

```bash
kubectl create deployment nginx --image=nginx
```

The Deployment will be created in:

```text
default
```

You can verify:

```bash
kubectl get deployments
```

Equivalent:

```bash
kubectl get deployments -n default
```

### Important

If your manifest does not specify a Namespace, Kubernetes generally creates the resource in the current/default Namespace used by the command.

Example:

```yaml
apiVersion: v1
kind: Pod

metadata:
  name: nginx

spec:
  containers:
    - name: nginx
      image: nginx
```

Apply:

```bash
kubectl apply -f pod.yaml
```

The Pod will normally be created in the `default` Namespace.

---

# 5. `kube-node-lease`

`kube-node-lease` is a Namespace used by Kubernetes for **Node Lease objects**.

Node Lease objects help Kubernetes determine whether a Node is healthy/alive.

Think of it as a heartbeat:

```text
Node
 │
 │ Heartbeat / Lease
 ↓
Kubernetes Control Plane
```

If Kubernetes stops receiving the expected heartbeat from a Node, it can determine that the Node may have become unavailable.

You can see the leases using:

```bash
kubectl get lease -n kube-node-lease
```

Example:

```text
NAME       HOLDER       AGE
minikube   minikube     ...
```

### Simple way to remember

```text
kube-node-lease
        ↓
Node heartbeat / health information
```

---

# 6. `kube-public`

`kube-public` is intended for resources that should be publicly readable across the cluster.

For example:

```text
kube-public
     │
     └── Cluster-wide publicly readable information
```

You can inspect it:

```bash
kubectl get all -n kube-public
```

Important:

> `kube-public` does NOT mean publicly accessible from the Internet.

It means resources in this Namespace can be readable by cluster users under appropriate Kubernetes permissions.

---

# 7. `kube-system`

`kube-system` contains resources created by Kubernetes itself and/or cluster components.

Examples may include:

```text
CoreDNS
kube-proxy
CNI components
Metrics components
Other cluster services
```

Check:

```bash
kubectl get pods -n kube-system
```

Example:

```text
NAME                         READY   STATUS
coredns-xxxxx                1/1     Running
kube-proxy-xxxxx             1/1     Running
```

### Important

Avoid deleting or modifying resources in `kube-system` unless you understand exactly what they do.

---

# 8. Creating a Namespace

## Using kubectl

Create a Namespace called `nginx`:

```bash
kubectl create namespace nginx
```

Short form:

```bash
kubectl create ns nginx
```

Verify:

```bash
kubectl get namespaces
```

or:

```bash
kubectl get ns
```

---

# 9. Creating a Namespace Using YAML

Namespaces can also be created using a manifest file.

Create:

```text
namespace.yaml
```

Content:

```yaml
apiVersion: v1
kind: Namespace

metadata:
  name: dev
```

Apply:

```bash
kubectl apply -f namespace.yaml
```

Verify:

```bash
kubectl get ns
```

Expected:

```text
NAME     STATUS
default  Active
dev      Active
```

---

# 10. Deploying a Pod into a Namespace

There are multiple ways to specify the Namespace.

## Method 1 — Namespace in the Manifest

Example:

```yaml
apiVersion: v1
kind: Pod

metadata:
  name: nginx
  namespace: dev

spec:
  containers:
    - name: nginx
      image: nginx
```

Apply:

```bash
kubectl apply -f nginx.yaml
```

The Pod will be created in:

```text
dev
```

Check:

```bash
kubectl get pods -n dev
```

---

# 11. Namespace in Different Resources

A Namespace is normally specified under:

```yaml
metadata:
  namespace: <namespace-name>
```

Example:

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: nginx-deployment
  namespace: dev

spec:
  replicas: 2

  selector:
    matchLabels:
      app: nginx

  template:
    metadata:
      labels:
        app: nginx

    spec:
      containers:
        - name: nginx
          image: nginx
          ports:
            - containerPort: 80
```

Apply:

```bash
kubectl apply -f deployment.yaml
```

Check:

```bash
kubectl get deployment -n dev
```

Pods:

```bash
kubectl get pods -n dev
```

---

# 12. Deploying Without Putting Namespace in YAML

You can specify the Namespace from the command line.

Example:

```bash
kubectl apply -f deployment.yaml -n dev
```

This tells Kubernetes:

```text
Apply deployment.yaml
        ↓
dev Namespace
```

This is useful when the same YAML needs to be deployed to multiple environments.

For example:

```bash
kubectl apply -f deployment.yaml -n dev
```

and:

```bash
kubectl apply -f deployment.yaml -n test
```

and:

```bash
kubectl apply -f deployment.yaml -n prod
```

---

# 13. Checking Resources in a Namespace

Pods:

```bash
kubectl get pods -n dev
```

Deployments:

```bash
kubectl get deployments -n dev
```

Services:

```bash
kubectl get services -n dev
```

All common resources:

```bash
kubectl get all -n dev
```

---

# 14. Checking Resources Across All Namespaces

To see Pods from every Namespace:

```bash
kubectl get pods -A
```

or:

```bash
kubectl get pods --all-namespaces
```

Example:

```text
NAMESPACE     NAME
default       nginx
dev           backend
dev           frontend
prod          backend
kube-system   coredns
```

This is a very useful command when troubleshooting.

---

# 15. Getting a Specific Resource from a Namespace

Example:

```bash
kubectl get pod nginx -n dev
```

Describe:

```bash
kubectl describe pod nginx -n dev
```

Logs:

```bash
kubectl logs nginx -n dev
```

Delete:

```bash
kubectl delete pod nginx -n dev
```

---

# 16. Current Namespace

Kubernetes commands use the current context/Namespace.

Check your current context:

```bash
kubectl config current-context
```

You can inspect the complete configuration:

```bash
kubectl config view
```

By default, commands normally operate against the `default` Namespace unless another Namespace is specified/configured.

---

# 17. Setting a Default Namespace for the Current Context

Instead of repeatedly writing:

```bash
kubectl get pods -n dev
kubectl get deployments -n dev
kubectl get services -n dev
```

you can configure the Namespace for the current context.

```bash
kubectl config set-context --current --namespace=dev
```

Now:

```bash
kubectl get pods
```

means:

```bash
kubectl get pods -n dev
```

Check the configured Namespace:

```bash
kubectl config view --minify --output 'jsonpath={..namespace}'
```

---

# 18. Important Difference: Namespace vs Cluster

A Namespace is **not a separate Kubernetes cluster**.

Example:

```text
                 Kubernetes Cluster
                       │
        ┌──────────────┼──────────────┐
        │              │              │
       dev            test           prod
        │              │              │
     resources      resources      resources
```

All three Namespaces still use the same underlying cluster resources:

```text
Nodes
CPU
Memory
Network
Storage
Control Plane
```

Namespaces provide logical isolation and organization.

---

# 19. Namespace Isolation

Namespaces provide logical separation, but they do not automatically isolate network traffic.

For example:

```text
dev namespace
    │
    │ Network
    ↓
prod namespace
```

By default, Pods may be able to communicate across Namespaces depending on the cluster/network configuration.

If you need to restrict communication, use:

```text
NetworkPolicy
```

Example concept:

```text
dev → dev     ✅
dev → test    ❌
dev → prod    ❌
```

NetworkPolicy is used to implement this kind of network isolation.

---

# 20. ResourceQuota

A ResourceQuota limits the total resources that can be consumed in a Namespace.

Example:

```yaml
apiVersion: v1
kind: ResourceQuota

metadata:
  name: dev-quota
  namespace: dev

spec:
  hard:
    pods: "10"
    requests.cpu: "2"
    requests.memory: 4Gi
    limits.cpu: "4"
    limits.memory: 8Gi
```

Apply:

```bash
kubectl apply -f resource-quota.yaml
```

Check:

```bash
kubectl get resourcequota -n dev
```

Detailed information:

```bash
kubectl describe resourcequota dev-quota -n dev
```

---

# 21. ResourceQuota vs Namespace

These are different concepts.

### Namespace

Provides logical organization:

```text
dev
test
prod
```

### ResourceQuota

Controls resource usage:

```text
dev
 ├── Maximum Pods: 10
 ├── CPU: 4 cores
 └── Memory: 8 Gi
```

So:

```text
Namespace
   +
ResourceQuota
   +
RBAC
   +
NetworkPolicy
   =
Better isolation and governance
```

---

# 22. RBAC and Namespace Access

RBAC can restrict users to specific Namespaces.

Example:

```text
Dev Team
    │
    └── RoleBinding
            │
            ↓
        dev Namespace
```

A developer may have permission to:

```text
get pods
create deployments
update services
```

inside `dev`, while having no permissions in `prod`.

Typical RBAC resources:

```text
Role
RoleBinding
ClusterRole
ClusterRoleBinding
```

Remember:

```text
Role
   ↓
Permissions inside a Namespace

RoleBinding
   ↓
Assigns the Role to a user/group/service account
```

---

# 23. Namespaced vs Cluster-Scoped Resources

Not every Kubernetes resource belongs to a Namespace.

### Namespaced resources

Examples:

```text
Pod
Deployment
Service
ConfigMap
Secret
Role
RoleBinding
ResourceQuota
```

These can exist separately in different Namespaces.

Example:

```text
dev/nginx
prod/nginx
```

### Cluster-scoped resources

Examples:

```text
Node
Namespace
PersistentVolume
ClusterRole
ClusterRoleBinding
StorageClass
```

These belong to the entire cluster.

For example:

```bash
kubectl get nodes
```

You don't use:

```bash
kubectl get nodes -n dev
```

because Nodes are cluster-scoped resources.

---

# 24. Useful Namespace Commands

### List Namespaces

```bash
kubectl get namespaces
```

### Create Namespace

```bash
kubectl create namespace dev
```

### Delete Namespace

```bash
kubectl delete namespace dev
```

⚠️ Be careful:

Deleting a Namespace deletes the resources inside it.

```text
delete dev namespace
        ↓
Pods        ❌
Deployments ❌
Services    ❌
ConfigMaps  ❌
Secrets     ❌
```

### Get Pods in Namespace

```bash
kubectl get pods -n dev
```

### Get Everything

```bash
kubectl get all -n dev
```

### Get Resources in Every Namespace

```bash
kubectl get pods -A
```

### Describe Namespace

```bash
kubectl describe namespace dev
```

---

# 25. Practical Example — Dev and Prod

Let's create two Namespaces:

```bash
kubectl create namespace dev
kubectl create namespace prod
```

Verify:

```bash
kubectl get ns
```

Create the same Deployment in both:

```bash
kubectl create deployment nginx \
  --image=nginx \
  -n dev
```

```bash
kubectl create deployment nginx \
  --image=nginx \
  -n prod
```

Now:

```bash
kubectl get deployments -n dev
```

Output:

```text
NAME    READY
nginx   1/1
```

And:

```bash
kubectl get deployments -n prod
```

Output:

```text
NAME    READY
nginx   1/1
```

The Deployment names are the same because they exist in different Namespaces.

---

# 26. Real-World Environment Structure

A common structure can look like:

```text
Kubernetes Cluster
│
├── dev
│   ├── frontend
│   ├── backend
│   └── database
│
├── test
│   ├── frontend
│   ├── backend
│   └── database
│
├── staging
│   ├── frontend
│   ├── backend
│   └── database
│
└── prod
    ├── frontend
    ├── backend
    └── database
```

This allows teams to organize resources by environment.

---

# 27. Important YAML Structure to Remember

Basic Namespace manifest:

```yaml
apiVersion: v1
kind: Namespace

metadata:
  name: dev
```

Basic Pod deployed into Namespace:

```yaml
apiVersion: v1
kind: Pod

metadata:
  name: nginx
  namespace: dev

spec:
  containers:
    - name: nginx
      image: nginx
```

The important part is:

```yaml
metadata:
  namespace: dev
```

---

# 28. Common Mistakes

### Mistake 1 — Namespace doesn't exist

If your YAML says:

```yaml
metadata:
  namespace: dev
```

but `dev` doesn't exist, the resource cannot be created.

Create it first:

```bash
kubectl create namespace dev
```

---

### Mistake 2 — Checking the wrong Namespace

You created:

```text
dev/nginx
```

but run:

```bash
kubectl get pods
```

You may see:

```text
No resources found
```

because this command checks the current/default Namespace.

Use:

```bash
kubectl get pods -n dev
```

---

### Mistake 3 — Assuming Namespace provides complete isolation

Namespace ≠ complete security isolation.

For stronger isolation, Kubernetes commonly uses:

```text
Namespaces
    +
RBAC
    +
ResourceQuota
    +
LimitRange
    +
NetworkPolicy
```

---

# 29. Namespace Mental Model

Remember it like this:

```text
                 Kubernetes Cluster
                         │
             ┌───────────┼───────────┐
             │           │           │
            DEV         TEST        PROD
             │           │           │
          Resources   Resources   Resources
             │           │           │
             └───────────┼───────────┘
                         │
              Shared Cluster Resources
```

Namespace = **logical boundary for organizing Kubernetes resources.**

---

# 30. Quick Revision

### What is a Namespace?

A logical partition used to organize Kubernetes resources within a cluster.

### Why use Namespaces?

```text
1. Resource organization
2. Same resource names in different environments
3. Resource quotas
4. RBAC/access control
5. Environment separation
6. Easier management
```

### Default Namespaces

```text
default
kube-node-lease
kube-public
kube-system
```

### Create Namespace

```bash
kubectl create ns dev
```

### YAML

```yaml
apiVersion: v1
kind: Namespace

metadata:
  name: dev
```

### Deploy into Namespace

```bash
kubectl apply -f deployment.yaml -n dev
```

### Check Namespace

```bash
kubectl get ns
```

### Check Pods

```bash
kubectl get pods -n dev
```

### Check all Namespaces

```bash
kubectl get pods -A
```

### Set current Namespace

```bash
kubectl config set-context --current --namespace=dev
```

---

# 31. Hands-On Practice

Try these exercises yourself.

## Exercise 1 — Create Namespaces

Create:

```text
dev
test
prod
```

Commands:

```bash
kubectl create ns dev
kubectl create ns test
kubectl create ns prod
```

Verify:

```bash
kubectl get ns
```

---

## Exercise 2 — Deploy Nginx

Deploy nginx into `dev`:

```bash
kubectl create deployment nginx \
  --image=nginx \
  -n dev
```

Check:

```bash
kubectl get deployment -n dev
kubectl get pods -n dev
```

---

## Exercise 3 — Same Name, Different Namespace

Deploy another nginx:

```bash
kubectl create deployment nginx \
  --image=nginx \
  -n prod
```

Check:

```bash
kubectl get deployments -n dev
kubectl get deployments -n prod
```

Understand why both can have:

```text
nginx
```

as their name.

---

## Exercise 4 — Explore kube-system

Run:

```bash
kubectl get pods -n kube-system
```

Identify:

- CoreDNS
- kube-proxy
- Other cluster components

---

## Exercise 5 — Practice ResourceQuota

Create a `dev` ResourceQuota that limits:

```text
Maximum Pods = 5
CPU = 2 cores
Memory = 4Gi
```

Then verify:

```bash
kubectl describe resourcequota -n dev
```

---

# 32. Interview Questions

### Q1. What is a Namespace?

A logical partition within a Kubernetes cluster used to organize and isolate resources.

### Q2. Why do we use Namespaces?

For resource organization, environment separation, resource quotas, and access control using RBAC.

### Q3. Can two Pods have the same name?

Yes, if they belong to different Namespaces.

Example:

```text
dev/nginx
prod/nginx
```

### Q4. Does Namespace create a separate cluster?

No. Multiple Namespaces exist inside the same Kubernetes cluster.

### Q5. What is `kube-system`?

A Namespace containing Kubernetes system components and other cluster-level services.

### Q6. What is `kube-node-lease`?

A Namespace containing Node Lease objects used to help track Node heartbeats/health.

### Q7. What is `kube-public`?

A Namespace intended for resources that should be readable across the cluster according to Kubernetes permissions.

### Q8. How do you create a Namespace?

```bash
kubectl create namespace dev
```

or using YAML:

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: dev
```

### Q9. How do you deploy a resource into a Namespace?

Using YAML:

```yaml
metadata:
  namespace: dev
```

or CLI:

```bash
kubectl apply -f deployment.yaml -n dev
```

### Q10. How do you see Pods from all Namespaces?

```bash
kubectl get pods -A
```

### Q11. Does Namespace provide network isolation?

Not by itself. Network isolation is normally implemented using NetworkPolicies.

### Q12. How do you restrict users from accessing a Namespace?

Use Kubernetes RBAC with resources such as:

```text
Role
RoleBinding
ClusterRole
ClusterRoleBinding
```

---

# 33. My Learning Checklist

- [ ] Understand what a Namespace is
- [ ] Understand why Namespaces are used
- [ ] Create Namespace using `kubectl`
- [ ] Create Namespace using YAML
- [ ] Deploy Pods into a Namespace
- [ ] Deploy Deployments into a Namespace
- [ ] Understand `default`
- [ ] Understand `kube-system`
- [ ] Understand `kube-public`
- [ ] Understand `kube-node-lease`
- [ ] Use `kubectl get pods -n <namespace>`
- [ ] Use `kubectl get pods -A`
- [ ] Set the current Namespace
- [ ] Understand ResourceQuota
- [ ] Understand RBAC with Namespaces
- [ ] Understand NetworkPolicy vs Namespace
- [ ] Understand Namespaced vs Cluster-scoped resources

---

# 34. One-Line Memory Trick

```text
Namespace = Organize

ResourceQuota = Limit

RBAC = Access

NetworkPolicy = Network Isolation
```

Together:

```text
Namespace
   │
   ├── Organization
   ├── ResourceQuota → How much?
   ├── RBAC          → Who can access?
   └── NetworkPolicy → Who can communicate?
```