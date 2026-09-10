# Minikube & Kubernetes Learning Notes

Simple notes for setting up Docker, Minikube, and working with a local Kubernetes cluster.

## 1. Docker Installation

Docker is used as the container runtime/driver for the Minikube cluster.

### Documentation

- Docker Desktop: https://www.docker.com/products/docker-desktop/
- Docker Documentation: https://docs.docker.com/

After installation, verify Docker:

```bash
docker --version
```

---

## 2. Minikube Installation

Minikube allows us to run a Kubernetes cluster locally.

### Documentation

- Minikube: https://minikube.sigs.k8s.io/docs/
- Minikube Installation: https://minikube.sigs.k8s.io/docs/start/

Verify the installation:

```bash
minikube version
```

Also verify `kubectl`:

```bash
kubectl version --client
```

---

## 3. Create a Local Cluster

Create a Minikube cluster with the name `local-cluster` using Docker as the driver:

```bash
minikube start --profile local-cluster --driver=docker
```

> Note: Minikube uses **profiles** to maintain multiple clusters.  
> The profile name here is `local-cluster`.

Check the nodes:

```bash
kubectl get nodes
```

Example:

```text
NAME             STATUS   ROLES           AGE   VERSION
local-cluster    Ready    control-plane   ...
```

---

## 4. Check Kubernetes Contexts

To see the Kubernetes clusters/contexts configured on the local machine:

```bash
kubectl config get-contexts
```

The output shows the available contexts and which context is currently selected.

Example:

```text
CURRENT   NAME            CLUSTER         AUTHINFO
*         local-cluster   local-cluster   ...
          minikube        minikube        ...
```

---

## 5. Select the Cluster / Context

To switch the current Kubernetes context:

```bash
kubectl config use-context local-cluster
```

This selects `local-cluster` as the cluster on which `kubectl` commands will operate.

### Important

The command:

```bash
kubectl set-context -p local-cluster
```

is **not a valid `kubectl` command**.

The correct command is:

```bash
kubectl config use-context local-cluster
```

You can verify the current context with:

```bash
kubectl config current-context
```

---

## 6. Add a Worker Node

Add a worker node to the `local-cluster` Minikube cluster:

```bash
minikube node add --worker -p local-cluster
```

Check the nodes:

```bash
kubectl get nodes
```

You should now see the control-plane node and the newly added worker node.

---

## 7. Delete a Worker Node

To delete a specific node:

```bash
minikube node delete local-cluster-m03 -p local-cluster
```

Replace `local-cluster-m03` with the actual node name you want to delete.

Verify:

```bash
kubectl get nodes
```

---

## 8. Open the Minikube Dashboard

To get the local URL for the Kubernetes dashboard:

```bash
minikube dashboard --url -p local-cluster
```

Minikube will return a local URL that can be opened in a browser.

Example:

```text
http://127.0.0.1:xxxxx
```

The dashboard provides a graphical view of resources running inside the Kubernetes cluster.

---

## 9. Quick Command Reference

| Task | Command |
|---|---|
| Check Docker | `docker --version` |
| Check Minikube | `minikube version` |
| Check kubectl | `kubectl version --client` |
| Create cluster | `minikube start --profile local-cluster --driver=docker` |
| Get nodes | `kubectl get nodes` |
| List contexts | `kubectl config get-contexts` |
| Switch context | `kubectl config use-context local-cluster` |
| Check current context | `kubectl config current-context` |
| Add worker node | `minikube node add --worker -p local-cluster` |
| Delete node | `minikube node delete <node-name> -p local-cluster` |
| Dashboard URL | `minikube dashboard --url -p local-cluster` |

---

## Learning Progress

- [x] Docker installation
- [x] Minikube installation
- [x] Create a Minikube cluster
- [x] Use Docker as the Minikube driver
- [x] Check Kubernetes nodes
- [x] Understand Kubernetes contexts
- [x] Switch between clusters/contexts
- [x] Add worker nodes
- [x] Delete worker nodes
- [x] Access the Minikube dashboard