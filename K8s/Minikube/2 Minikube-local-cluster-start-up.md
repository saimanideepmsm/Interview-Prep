# Minikube Local Cluster — Startup Guide

This setup uses **Minikube with Docker** and a custom Minikube profile named:

```text
local-cluster
```

The cluster contains:

* `local-cluster` — Control Plane
* `local-cluster-m02` — Worker Node

---

## 🚀 Starting the Cluster After Laptop Restart

After restarting your laptop, open **CMD or PowerShell**.

### 1. Check Minikube Status

Run:

```cmd
minikube status -p local-cluster
```

If you see:

```text
local-cluster
type: Control Plane
host: Running
kubelet: Running
apiserver: Running
kubeconfig: Configured

local-cluster-m02
type: Worker
host: Running
kubelet: Running
```

The cluster is already running.

✅ You can start using Kubernetes immediately.

---

## 2. If the Cluster Is Stopped

If the status shows that the cluster is stopped, start it with:

```cmd
minikube start -p local-cluster
```

Wait for Minikube to finish starting.

Then verify:

```cmd
minikube status -p local-cluster
```

---

## 3. Verify Kubernetes Nodes

Run:

```cmd
kubectl get nodes
```

Expected output:

```text
NAME                STATUS   ROLES           AGE     VERSION
local-cluster       Ready    control-plane   ...     v1.37.0
local-cluster-m02   Ready    <none>          ...     v1.37.0
```

Both nodes should have:

```text
STATUS = Ready
```

---

# 🔧 Useful Commands

## Check Minikube Status

```cmd
minikube status -p local-cluster
```

## Start Cluster

```cmd
minikube start -p local-cluster
```

## Stop Cluster

```cmd
minikube stop -p local-cluster
```

Stopping the cluster does **not** delete it.

## Delete Cluster

⚠️ This removes the Minikube cluster:

```cmd
minikube delete -p local-cluster
```

Do **not** use this for normal shutdown/startup.

---

# ☸️ Useful Kubernetes Commands

### Check Nodes

```cmd
kubectl get nodes
```

### Check All Pods

```cmd
kubectl get pods -A
```

### Check Services

```cmd
kubectl get svc
```

### Check Current Context

```cmd
kubectl config current-context
```

Expected:

```text
local-cluster
```

### Check Available Contexts

```cmd
kubectl config get-contexts
```

---

# ⚠️ Important: Profile Name

This cluster uses the custom profile:

```text
local-cluster
```

Therefore, use:

```cmd
minikube status -p local-cluster
minikube start -p local-cluster
minikube stop -p local-cluster
```

Do **not** use:

```cmd
minikube status
minikube start
minikube stop
```

Those commands target Minikube's default profile named:

```text
minikube
```

which is not the profile used by this setup.

---

# 🧹 Laptop Shutdown

You don't need to manually delete or recreate anything before shutting down your laptop.

You can simply:

```text
Use Kubernetes
    ↓
Shut down / Restart laptop
    ↓
Open CMD / PowerShell
    ↓
minikube status -p local-cluster
    ↓
If stopped:
minikube start -p local-cluster
    ↓
kubectl get nodes
    ↓
Start working
```

---

# 🆘 If Something Goes Wrong

### `Profile "minikube" not found`

You probably forgot the profile name.

Use:

```cmd
minikube status -p local-cluster
```

---

### `Unable to connect to the server`

First check:

```cmd
minikube status -p local-cluster
```

If the cluster isn't running:

```cmd
minikube start -p local-cluster
```

Then:

```cmd
kubectl get nodes
```

---

### Check Minikube Profiles

```cmd
minikube profile list
```

You should see:

```text
local-cluster
```

---

# ⭐ Quick Startup Cheat Sheet

For normal daily use, these are the only commands you usually need:

```cmd
minikube status -p local-cluster
```

If stopped:

```cmd
minikube start -p local-cluster
```

Then:

```cmd
kubectl get nodes
```

If both nodes show `Ready`, you're ready to go! 🚀
