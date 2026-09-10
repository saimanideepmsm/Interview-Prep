# Kubernetes – Pods & Basic kubectl Commands

This README covers some basic Kubernetes concepts and commands learned while working with Pods.

---

## 1. Pod and Pod Seeds

### What is a Pod?

A **Pod** is the smallest deployable unit in Kubernetes.

A Pod can contain **one or more containers** that are deployed and managed together.

Think of it like:

```text
Kubernetes
   │
   └── Pod
        │
        ├── Container
        ├── Resources
        ├── Volumes
        └── Networking
```

### What is a Pod Seed?

A **Pod seed** is not a standard Kubernetes concept.

If you meant **Pod** and **Pod spec**, the **Pod specification (`spec`)** defines how the Pod should run, including:

* Containers
* Container images
* Ports
* Resources
* Volumes
* Environment variables
* Restart behavior
* Networking-related configuration

Example:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod

spec:
  containers:
    - name: nginx
      image: nginx
```

---

# 2. Pod as a Group of Containers, Resources and Volumes

A Pod acts as a **logical group** for one or more containers.

Containers inside the same Pod share certain resources.

### Containers

A Pod can contain multiple containers:

```text
Pod
│
├── Container 1
│
└── Container 2
```

This is useful when containers need to work closely together.

For example:

```text
Pod
│
├── Application Container
│
└── Logging/Monitoring Container
```

These containers share the same Pod environment.

### Resources

Kubernetes allows you to define resources such as:

* CPU
* Memory

Example:

```yaml
resources:
  requests:
    cpu: "100m"
    memory: "128Mi"

  limits:
    cpu: "500m"
    memory: "512Mi"
```

**Requests** indicate the resources required by the container.

**Limits** specify the maximum resources the container can consume.

### Volumes

Volumes provide storage that can be accessed by containers inside a Pod.

Example:

```text
Pod
│
├── Container
│
└── Volume
```

Volumes are useful when containers need to share or persist data.

---

# 3. Container Connectivity

Containers inside the **same Pod** can communicate with each other using the Pod's networking environment.

They can use:

```text
localhost
```

For example, suppose a Pod contains:

```text
Pod
│
├── Nginx Container
│      Port: 80
│
└── Application Container
       Port: 8080
```

The application container can communicate with Nginx using:

```text
localhost:80
```

### Important

`localhost` refers to the **same network namespace**.

Therefore:

```text
Container A ───── localhost ───── Container B
             (same Pod)
```

This works because containers in the same Pod share the Pod's network namespace.

### Different Pods

Containers in **different Pods** should not normally communicate using `localhost`.

For example:

```text
Pod A
└── Container A

Pod B
└── Container B
```

Here:

```text
localhost
```

inside Container A refers to Pod A, not Pod B.

Kubernetes Services are commonly used to provide stable networking between Pods.

---

# 4. Creating an Nginx Pod Using kubectl

Instead of creating a YAML file manually, Kubernetes allows us to quickly create a Pod using `kubectl run`.

## Create an Nginx Pod

```bash
kubectl run nginx-pod --image=nginx
```

### Explanation

```text
kubectl
   │
   └── run
        │
        ├── nginx-pod
        │
        └── --image=nginx
```

* `kubectl` → Kubernetes command-line tool
* `run` → creates a Pod
* `nginx-pod` → name of the Pod
* `--image=nginx` → uses the Nginx container image

---

## Check Pods

After creating the Pod:

```bash
kubectl get pods
```

Example output:

```text
NAME         READY   STATUS    RESTARTS   AGE
nginx-pod    1/1     Running   0          10s
```

### Useful Pod commands

Get Pods:

```bash
kubectl get pods
```

Get more detailed information:

```bash
kubectl get pods -o wide
```

Describe a Pod:

```bash
kubectl describe pod nginx-pod
```

---

## Delete the Pod

To delete the Nginx Pod:

```bash
kubectl delete pod nginx-pod
```

After deletion, verify:

```bash
kubectl get pods
```

The `nginx-pod` should no longer appear.

---

# 5. Finding Kubernetes API Resources

Kubernetes has many different resources/components, such as:

* Pods
* Services
* Deployments
* ConfigMaps
* Secrets
* Nodes
* Namespaces
* ReplicaSets

You can use `kubectl api-resources` to list the resources supported by your Kubernetes API server.

```bash
kubectl api-resources
```

Example:

```text
NAME         SHORTNAMES   APIVERSION   NAMESPACED   KIND
pods         po           v1           true         Pod
services     svc          v1           true         Service
deployments  deploy       apps/v1      true         Deployment
nodes         no          v1           false        Node
```

---

## Finding a Specific Resource

On Linux/Git Bash, you can filter the output using:

```bash
kubectl api-resources | grep pods
```

This helps you quickly find information related to Pods.

### Windows PowerShell

If you are using PowerShell, `grep` may not be available by default.

You can use:

```powershell
kubectl api-resources | Select-String pods
```

Or simply:

```powershell
kubectl api-resources
```

and search through the output.

---

# 6. API Version

Kubernetes resources belong to an **API group and version**.

For example:

```text
Pod
└── v1
```

A Deployment commonly uses:

```text
Deployment
└── apps/v1
```

This is why Kubernetes YAML files contain:

```yaml
apiVersion: v1
```

or:

```yaml
apiVersion: apps/v1
```

Example Pod:

```yaml
apiVersion: v1
kind: Pod
```

Example Deployment:

```yaml
apiVersion: apps/v1
kind: Deployment
```

The API version tells Kubernetes which API schema should be used to interpret the resource definition.

---

# 7. Quick Command Reference

| Command                                       | Purpose                                      |
| --------------------------------------------- | -------------------------------------------- |
| `kubectl get pods`                            | List Pods                                    |
| `kubectl run nginx-pod --image=nginx`         | Create an Nginx Pod                          |
| `kubectl describe pod nginx-pod`              | Show detailed Pod information                |
| `kubectl get pods -o wide`                    | Show additional Pod details                  |
| `kubectl delete pod nginx-pod`                | Delete a Pod                                 |
| `kubectl api-resources`                       | List Kubernetes API resources                |
| `kubectl api-resources \| grep pods`          | Find Pod-related resources on Linux/Git Bash |
| `kubectl api-resources \| Select-String pods` | Find Pod-related resources in PowerShell     |

---

# 8. Key Takeaways

```text
Pod
 │
 ├── Contains one or more containers
 ├── Can define CPU/Memory resources
 ├── Can use volumes for storage
 └── Provides a shared network environment
        │
        └── Containers in the same Pod
              can communicate using localhost
```

### Remember

1. **Pod is the smallest deployable unit in Kubernetes.**
2. A Pod can contain **one or multiple containers**.
3. Containers in the same Pod share the **network namespace**.
4. Containers in the same Pod can communicate using **localhost**.
5. `kubectl run` can quickly create a Pod.
6. `kubectl get pods` lists Pods.
7. `kubectl delete pod` removes a Pod.
8. `kubectl api-resources` lists resources supported by the Kubernetes API server.
9. Kubernetes resources have an **API version**, such as `v1` or `apps/v1`.
10. In PowerShell, use `Select-String` instead of `grep`.

---

## Practice

Try the following sequence:

```bash
kubectl run nginx-pod --image=nginx

kubectl get pods

kubectl describe pod nginx-pod

kubectl get pods -o wide

kubectl api-resources

kubectl delete pod nginx-pod

kubectl get pods
```

This gives you a simple hands-on workflow for creating, inspecting, and deleting a Kubernetes Pod.
