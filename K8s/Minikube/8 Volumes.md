# Kubernetes Volumes & Persistent Storage

## 1. The Problem — What Happens to Data When a Pod Dies?

One of the important characteristics of Kubernetes is that **Pods are ephemeral**.

Consider a Deployment:

```text
Deployment
   │
   ├── Pod 1
   ├── Pod 2
   └── Pod 3
```

Suppose Pod 1 stores some data:

```text
Pod 1
└── /app/data
      ├── file1.txt
      └── file2.txt
```

Now Pod 1 crashes or is deleted.

Kubernetes creates a replacement:

```text
Old Pod 1 ❌
     ↓
New Pod 1 ✅
```

The new Pod normally gets a **new container filesystem**.

Therefore:

```text
Old Pod filesystem
      ↓
      ❌ Lost

New Pod filesystem
      ↓
      🆕 Empty
```

This is one of the major reasons Kubernetes needs **Volumes**.

---

# 2. Why Can't We Store Data Inside the Container?

A container has its own writable filesystem.

For example:

```text
Pod
│
└── Container
      │
      └── Filesystem
           └── /app/data
```

If the container is recreated, the filesystem is recreated as well.

Therefore:

```text
Container
    │
    └── Application data
             ↓
       Container dies
             ↓
       Data may be lost
```

For temporary application data this may be perfectly fine.

For important data such as:

```text
Database files
User uploads
Application state
Logs
Configuration generated at runtime
```

we generally need storage that exists **outside the container's lifecycle**.

---

# 3. What is a Kubernetes Volume?

A Kubernetes **Volume** provides storage that can be mounted into one or more containers in a Pod.

Conceptually:

```text
                Volume
                   │
          ┌────────┴────────┐
          ↓                 ↓
       Container 1       Container 2
          │                 │
          └───────┬─────────┘
                  ↓
              Shared Data
```

A Volume can be used for:

- Sharing data between containers
- Keeping data available when a container restarts
- Storing temporary application data
- Providing persistent storage

---

# 4. Important Concept — Pod vs Container

Volumes are associated with a **Pod**, not directly with an individual container.

Example:

```text
Pod
│
├── Container A
│
├── Container B
│
└── Volume
      │
      └── /shared-data
```

Both containers can mount the same Volume.

Example:

```text
Container A
     │
     ↓
 /shared-data
     ↑
     │
Container B
```

This allows containers within the same Pod to share files.

---

# 5. `emptyDir` — Ephemeral Volume

One simple Kubernetes Volume type is:

```text
emptyDir
```

`emptyDir` creates an empty directory when a Pod is created.

Example:

```yaml
apiVersion: v1
kind: Pod

metadata:
  name: volume-demo

spec:
  containers:

    - name: container-1
      image: nginx

      volumeMounts:
        - name: shared-storage
          mountPath: /data

    - name: container-2
      image: busybox

      volumeMounts:
        - name: shared-storage
          mountPath: /data

  volumes:
    - name: shared-storage
      emptyDir: {}
```

---

# 6. How `emptyDir` Works

The structure looks like:

```text
Pod
│
├── Container 1
│      │
│      └── /data
│
├── Container 2
│      │
│      └── /data
│
└── emptyDir
       │
       └── Shared directory
```

If Container 1 writes:

```text
/data/test.txt
```

Container 2 can read:

```text
/data/test.txt
```

because both containers mount the same volume.

---

# 7. Important Limitation of `emptyDir`

`emptyDir` survives a **container restart**, but it does **not survive Pod deletion**.

Example:

```text
Pod
│
├── Container
│
└── emptyDir
```

Container crashes:

```text
Container ❌
     ↓
Container restarted
     ↓
emptyDir still exists ✅
```

But if the Pod itself is deleted:

```text
Pod ❌
 │
 └── emptyDir ❌
```

A new Pod gets a new `emptyDir`.

```text
Old Pod
   ↓
deleted ❌

New Pod
   ↓
new emptyDir 🆕
```

Therefore:

```text
emptyDir = temporary / ephemeral storage
```

---

# 8. Sharing Data Between Containers vs Persistent Storage

These are two different problems.

### Problem 1 — Containers inside the same Pod need to share data

Use:

```text
emptyDir
```

Example:

```text
Pod
├── Container A
├── Container B
└── emptyDir
```

### Problem 2 — Data must survive Pod deletion

Use persistent storage such as:

```text
PersistentVolume
PersistentVolumeClaim
StorageClass
```

---

# 9. `hostPath` Volume

Another volume type is:

```text
hostPath
```

Instead of keeping data only inside the Pod, we mount a directory from the Kubernetes Node.

Conceptually:

```text
Kubernetes Node
│
└── /data/myapp
        │
        ↓
      Volume
        │
        ↓
       Pod
```

Example:

```yaml
apiVersion: v1
kind: Pod

metadata:
  name: hostpath-demo

spec:
  containers:

    - name: nginx
      image: nginx

      volumeMounts:
        - name: host-storage
          mountPath: /data

  volumes:
    - name: host-storage

      hostPath:
        path: /data/myapp
        type: DirectoryOrCreate
```

---

# 10. How `hostPath` Works

Suppose the Pod is running on:

```text
Node 1
```

and the Node contains:

```text
/data/myapp
```

Then:

```text
Node 1
│
└── /data/myapp
        │
        ↓
     hostPath
        │
        ↓
       Pod
        │
        └── /data
```

Anything written to:

```text
/data
```

inside the container is stored in:

```text
/data/myapp
```

on the Node.

---

# 11. Problem With `hostPath`

`hostPath` is tied to a particular Node.

Imagine:

```text
Node 1
└── /data/myapp
```

Pod is running here:

```text
Pod A → Node 1
```

Now Node 1 fails.

Kubernetes may schedule the replacement Pod on:

```text
Node 2
```

But:

```text
Node 2
└── /data/myapp
```

may not contain the original data.

Therefore:

```text
hostPath
    ↓
Node-specific storage
    ↓
Node failure/deletion
    ↓
Data may be unavailable/lost
```

This is why `hostPath` is generally not the preferred solution for production persistent application data.

---

# 12. The Persistent Storage Solution

For data that should survive:

```text
Container restart
Pod restart
Pod recreation
Pod rescheduling
Node changes
```

we need persistent storage.

Kubernetes provides concepts such as:

```text
PersistentVolume (PV)
        │
        ↓
PersistentVolumeClaim (PVC)
        │
        ↓
Pod / Deployment
```

And:

```text
StorageClass
```

can be used to dynamically provision storage.

---

# 13. PersistentVolume (PV)

A **PersistentVolume (PV)** is a Kubernetes resource representing storage available to the cluster.

Think of it as:

```text
PV = Storage resource available to Kubernetes
```

Example:

```yaml
apiVersion: v1
kind: PersistentVolume

metadata:
  name: my-pv

spec:
  capacity:
    storage: 5Gi

  accessModes:
    - ReadWriteOnce

  hostPath:
    path: /mnt/data
```

Check PVs:

```bash
kubectl get pv
```

Detailed information:

```bash
kubectl describe pv my-pv
```

---

# 14. PersistentVolumeClaim (PVC)

A **PersistentVolumeClaim** is a request for storage by an application.

Think of it as:

```text
PV  = Available storage

PVC = Application's request for storage
```

Example:

```yaml
apiVersion: v1
kind: PersistentVolumeClaim

metadata:
  name: my-pvc

spec:
  accessModes:
    - ReadWriteOnce

  resources:
    requests:
      storage: 5Gi
```

Check:

```bash
kubectl get pvc
```

Detailed:

```bash
kubectl describe pvc my-pvc
```

---

# 15. PV and PVC Relationship

The basic relationship is:

```text
PersistentVolume
       │
       │ Provides storage
       ↓
PersistentVolumeClaim
       │
       │ Requests storage
       ↓
Pod / Deployment
```

A useful way to remember:

```text
PV = Storage

PVC = Request for Storage

Pod = Consumer of Storage
```

---

# 16. PV and PVC Binding

Suppose we have:

```text
PV
5Gi
```

and:

```text
PVC
requests 5Gi
```

Kubernetes can bind them:

```text
PV
5Gi
 │
 │ Bound
 ↓
PVC
5Gi
```

Check:

```bash
kubectl get pv
```

You may see:

```text
NAME     CAPACITY   ACCESS MODES   STATUS   CLAIM
my-pv    5Gi        RWO            Bound    default/my-pvc
```

---

# 17. Using PVC in a Pod

Once a PVC exists, the Pod can mount it.

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

      volumeMounts:
        - name: persistent-storage
          mountPath: /data

  volumes:

    - name: persistent-storage

      persistentVolumeClaim:
        claimName: my-pvc
```

The important part is:

```yaml
persistentVolumeClaim:
  claimName: my-pvc
```

---

# 18. Complete PV → PVC → Pod Flow

The complete flow is:

```text
                    Kubernetes Cluster
                           │
                           ↓
                PersistentVolume
                      5Gi
                           │
                           ↓
                PersistentVolumeClaim
                      5Gi
                           │
                           ↓
                         Pod
                           │
                           ↓
                    /data inside Pod
```

More simply:

```text
PV
 ↓
PVC
 ↓
Pod
```

---

# 19. Complete Example

## Step 1 — Create PV

```yaml
apiVersion: v1
kind: PersistentVolume

metadata:
  name: app-pv

spec:
  capacity:
    storage: 5Gi

  accessModes:
    - ReadWriteOnce

  persistentVolumeReclaimPolicy: Retain

  hostPath:
    path: /mnt/data
```

Apply:

```bash
kubectl apply -f pv.yaml
```

Check:

```bash
kubectl get pv
```

---

## Step 2 — Create PVC

```yaml
apiVersion: v1
kind: PersistentVolumeClaim

metadata:
  name: app-pvc

spec:
  accessModes:
    - ReadWriteOnce

  resources:
    requests:
      storage: 5Gi
```

Apply:

```bash
kubectl apply -f pvc.yaml
```

Check:

```bash
kubectl get pvc
```

---

## Step 3 — Create Pod

```yaml
apiVersion: v1
kind: Pod

metadata:
  name: app-pod

spec:

  containers:

    - name: nginx
      image: nginx

      volumeMounts:
        - name: app-storage
          mountPath: /data

  volumes:

    - name: app-storage

      persistentVolumeClaim:
        claimName: app-pvc
```

Apply:

```bash
kubectl apply -f pod.yaml
```

Check:

```bash
kubectl get pod
```

---

# 20. Testing Persistent Data

Enter the Pod:

```bash
kubectl exec -it app-pod -- bash
```

Create a file:

```bash
echo "Hello Kubernetes" > /data/test.txt
```

Verify:

```bash
cat /data/test.txt
```

Exit:

```bash
exit
```

Delete the Pod:

```bash
kubectl delete pod app-pod
```

Create the Pod again:

```bash
kubectl apply -f pod.yaml
```

Enter:

```bash
kubectl exec -it app-pod -- bash
```

Check:

```bash
cat /data/test.txt
```

The goal of persistent storage is that the data remains available independently of the Pod's lifecycle.

---

# 21. StorageClass

Manually creating PVs can become difficult in large environments.

For example:

```text
100 applications
        ↓
100 PVs?
        ↓
Manual management becomes difficult
```

Kubernetes provides:

```text
StorageClass
```

A StorageClass defines how storage can be dynamically provisioned.

Conceptually:

```text
Application
     │
     ↓
PVC
     │
     ↓
StorageClass
     │
     ↓
Storage Provider
     │
     ↓
Persistent Volume
```

---

# 22. Dynamic Provisioning

Without dynamic provisioning:

```text
Admin
  │
  ├── Create PV
  │
  └── Create storage manually
       ↓
      PVC
       ↓
      Pod
```

With dynamic provisioning:

```text
Developer
    │
    ↓
   PVC
    │
    ↓
StorageClass
    │
    ↓
Kubernetes provisions storage
    │
    ↓
   PV
    │
    ↓
   Pod
```

This is much more scalable.

---

# 23. StorageClass Example

A simplified StorageClass example:

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass

metadata:
  name: fast-storage

provisioner: example.com/storage

reclaimPolicy: Delete

volumeBindingMode: WaitForFirstConsumer
```

> The actual `provisioner` depends on your Kubernetes environment and storage platform. For example, AKS, EKS, GKE, Minikube, and on-premises Kubernetes can use different provisioners.

Check StorageClasses:

```bash
kubectl get storageclass
```

or:

```bash
kubectl get sc
```

---

# 24. PVC With StorageClass

Instead of manually creating a PV, a PVC can request storage through a StorageClass.

Example:

```yaml
apiVersion: v1
kind: PersistentVolumeClaim

metadata:
  name: app-pvc

spec:

  storageClassName: fast-storage

  accessModes:
    - ReadWriteOnce

  resources:
    requests:
      storage: 10Gi
```

The general flow becomes:

```text
PVC
 │
 │ storageClassName
 ↓
StorageClass
 │
 ↓
Dynamic Provisioning
 │
 ↓
PV
 │
 ↓
Pod
```

---

# 25. Access Modes

Persistent storage supports different access modes.

## ReadWriteOnce — RWO

```text
ReadWriteOnce
```

The volume can be mounted read-write by a node.

Commonly represented as:

```yaml
accessModes:
  - ReadWriteOnce
```

Remember:

```text
RWO = Read + Write + One Node
```

---

## ReadOnlyMany — ROX

```text
ReadOnlyMany
```

The volume can be mounted as read-only by multiple nodes, when supported by the storage backend.

```text
ROX = Read Only + Many
```

---

## ReadWriteMany — RWX

```text
ReadWriteMany
```

The volume can be mounted read-write by multiple nodes, when supported by the storage backend.

```text
RWX = Read + Write + Many
```

---

# 26. Important — Access Mode vs Number of Pods

Don't confuse:

```text
Pod count
```

with:

```text
Access mode
```

For example:

```text
Deployment
replicas: 3
```

does not automatically mean one `ReadWriteOnce` volume can be mounted read-write by all three Pods across different Nodes.

Whether multiple Pods can use the volume depends on:

- Access mode
- Storage backend
- Node placement
- CSI driver
- Kubernetes/storage implementation

---

# 27. PersistentVolume Reclaim Policy

A PV can have a reclaim policy.

Common options include:

```text
Retain
Delete
```

## Retain

```text
PVC deleted
    ↓
PV retained
    ↓
Data can be preserved for manual recovery/reuse
```

Example:

```yaml
persistentVolumeReclaimPolicy: Retain
```

## Delete

```text
PVC deleted
    ↓
PV/storage may be deleted
```

This behavior depends on the storage provisioner.

---

# 28. Important Terminology

### Volume

Storage mounted into a Pod.

### emptyDir

Temporary storage associated with the Pod.

### hostPath

Mounts a directory from the Node into the Pod.

### PersistentVolume

A Kubernetes resource representing persistent storage.

### PersistentVolumeClaim

A request for persistent storage.

### StorageClass

Defines how storage can be dynamically provisioned.

### CSI

Container Storage Interface.

CSI provides a standard way for Kubernetes to interact with storage systems.

---

# 29. Comparing Storage Options

| Storage | Survives Container Restart | Survives Pod Deletion | Node Independent |
|---|---|---|---|
| Container filesystem | ❌ | ❌ | ❌ |
| `emptyDir` | ✅ | ❌ | ❌ |
| `hostPath` | ✅ | ✅* | ❌ |
| PV/PVC with suitable backend | ✅ | ✅ | ✅* |

`*` Depends on the storage implementation and configuration.

The important production concept is:

```text
Pod lifecycle
     ≠
Persistent data lifecycle
```

---

# 30. Real-World Kubernetes Storage

In cloud Kubernetes environments, persistent storage is usually backed by external storage systems.

For example:

```text
Kubernetes
    │
    ↓
PVC
    │
    ↓
StorageClass
    │
    ↓
CSI Driver
    │
    ↓
Cloud Storage
```

For example, in Azure environments:

```text
AKS
 │
 ↓
PVC
 │
 ↓
StorageClass
 │
 ↓
Azure CSI Driver
 │
 ↓
Azure managed storage
```

The exact storage service depends on the required workload and StorageClass.

---

# 31. Important Architecture

Think about storage as layers:

```text
                    Application
                         │
                         ↓
                        Pod
                         │
                         ↓
                       Volume
                         │
                         ↓
                        PVC
                         │
                         ↓
                        PV
                         │
                         ↓
                  Storage Backend
```

With dynamic provisioning:

```text
Application
     ↓
    Pod
     ↓
    PVC
     ↓
StorageClass
     ↓
 CSI Driver
     ↓
Storage Backend
     ↓
    PV
```

---

# 32. Common Mistakes

## Mistake 1 — Using `emptyDir` for important database data

Bad idea:

```text
Database
   ↓
emptyDir
```

If the Pod is deleted:

```text
Pod deleted
    ↓
emptyDir deleted
    ↓
Data lost
```

For persistent database storage, use an appropriate persistent storage solution.

---

## Mistake 2 — Assuming `hostPath` is cluster-wide storage

`hostPath` belongs to a specific Node.

```text
Node 1
└── /data

Node 2
└── /data
```

These are different directories.

They don't automatically contain the same data.

---

## Mistake 3 — Thinking PVC itself stores the data

A PVC is a **claim/request**, not the physical storage itself.

Remember:

```text
PV = Storage

PVC = Request for Storage
```

---

## Mistake 4 — Forgetting the Namespace

PVCs are **Namespaced resources**.

For example:

```yaml
metadata:
  name: app-pvc
  namespace: dev
```

A Pod in `dev` must reference a PVC available in the same Namespace.

You cannot simply reference a PVC from another Namespace.

---

# 33. Useful Commands

### List Persistent Volumes

```bash
kubectl get pv
```

### List PersistentVolumeClaims

```bash
kubectl get pvc
```

### PVC in specific Namespace

```bash
kubectl get pvc -n dev
```

### Describe PV

```bash
kubectl describe pv <pv-name>
```

### Describe PVC

```bash
kubectl describe pvc <pvc-name>
```

### List StorageClasses

```bash
kubectl get storageclass
```

### Short form

```bash
kubectl get sc
```

### Check Pod Volume Configuration

```bash
kubectl describe pod <pod-name>
```

---

# 34. Troubleshooting PVC

If a PVC is stuck in:

```text
Pending
```

check:

```bash
kubectl get pvc
```

Then:

```bash
kubectl describe pvc <pvc-name>
```

Also check:

```bash
kubectl get pv
```

and:

```bash
kubectl get storageclass
```

Typical causes include:

```text
❌ No matching PV
❌ StorageClass doesn't exist
❌ Storage provisioner problem
❌ Requested capacity unavailable
❌ Unsupported access mode
❌ CSI/storage backend issue
```

---

# 35. Mental Model

Remember the hierarchy:

```text
Container
   ↓
Temporary filesystem

Pod
   ↓
Volumes

Node
   ↓
hostPath

Cluster
   ↓
PersistentVolume

Application
   ↓
PersistentVolumeClaim

Dynamic provisioning
   ↓
StorageClass
```

---

# 36. One-Line Memory Trick

```text
emptyDir
    ↓
Temporary sharing inside a Pod

hostPath
    ↓
Storage from a specific Node

PV
    ↓
Persistent storage resource

PVC
    ↓
Application's request for storage

StorageClass
    ↓
Automatic/dynamic storage provisioning
```

---

# 37. The Complete Picture

The evolution of Kubernetes storage can be remembered like this:

```text
1. Container filesystem
        ↓
   Data disappears when container lifecycle ends


2. emptyDir
        ↓
   Containers in the same Pod can share data
        ↓
   But data disappears when Pod is deleted


3. hostPath
        ↓
   Data stored on Node
        ↓
   But tied to that Node


4. PV + PVC
        ↓
   Persistent storage abstraction


5. StorageClass + PVC
        ↓
   Dynamic provisioning
        ↓
   Production-friendly storage management
```

---

# 38. Learning Checklist

- [ ] Understand why container filesystems are ephemeral
- [ ] Understand why Pods need Volumes
- [ ] Understand Pod vs Container storage
- [ ] Understand `emptyDir`
- [ ] Create an `emptyDir` volume
- [ ] Share an `emptyDir` between two containers
- [ ] Understand `hostPath`
- [ ] Understand why `hostPath` is Node-specific
- [ ] Understand PersistentVolume
- [ ] Create a PV
- [ ] Understand PersistentVolumeClaim
- [ ] Create a PVC
- [ ] Bind PV and PVC
- [ ] Mount PVC into a Pod
- [ ] Test persistent data
- [ ] Understand StorageClass
- [ ] Understand dynamic provisioning
- [ ] Understand RWO / ROX / RWX
- [ ] Understand PV reclaim policies
- [ ] Understand basic CSI concepts
- [ ] Troubleshoot a PVC stuck in `Pending`

---

# 39. Interview Questions

### Q1. Why do we need Kubernetes Volumes?

Containers have ephemeral filesystems. Volumes provide a way to store or share data beyond the normal container filesystem lifecycle.

### Q2. What is `emptyDir`?

`emptyDir` is an ephemeral Volume created when a Pod is assigned to a Node. It can be shared between containers in the same Pod and is deleted when the Pod is removed.

### Q3. What is `hostPath`?

`hostPath` mounts a directory from the Kubernetes Node into a Pod.

### Q4. What is the problem with `hostPath`?

The storage is tied to a particular Node, so it is generally unsuitable for portable, highly available application persistence.

### Q5. What is a PersistentVolume?

A PersistentVolume is a Kubernetes resource representing persistent storage available to the cluster.

### Q6. What is a PersistentVolumeClaim?

A PersistentVolumeClaim is a request made by an application for persistent storage.

### Q7. What is the difference between PV and PVC?

```text
PV  → Storage resource

PVC → Request for storage
```

### Q8. What is a StorageClass?

A StorageClass defines a class/type of storage and can enable dynamic provisioning of PersistentVolumes.

### Q9. Can multiple Pods use the same PVC?

It depends on the storage backend and access mode. For example, `ReadWriteMany` can support multiple nodes mounting a volume read-write when the backend supports it.

### Q10. What happens to `emptyDir` when a Pod is deleted?

The `emptyDir` and its contents are deleted along with the Pod.

### Q11. What happens if a container restarts while using `emptyDir`?

The `emptyDir` normally remains because the Pod still exists.

### Q12. What is the relationship between PV, PVC and StorageClass?

```text
PVC
 ↓
requests storage
 ↓
StorageClass
 ↓
provisions storage
 ↓
PV
 ↓
mounted by Pod
```

---

# 40. Final Revision Diagram

```text
                         Kubernetes
                              │
                              │
                 ┌────────────┴────────────┐
                 │                         │
          Ephemeral Storage          Persistent Storage
                 │                         │
          ┌──────┴──────┐          ┌───────┴────────┐
          │             │          │                │
      emptyDir      hostPath       PV              PVC
          │             │                           │
          │             │                           │
       Pod-level      Node-level                    │
       temporary      storage                       │
                                                    │
                                                    ↓
                                             StorageClass
                                                    │
                                                    ↓
                                            Dynamic Provisioning
                                                    │
                                                    ↓
                                             Storage Backend
```

## Golden Rule

> **If the data belongs to the Pod's temporary lifecycle → use an ephemeral Volume.**
>
> **If the data must survive Pod recreation → use persistent storage.**

And remember:

```text
PV  = What storage exists?

PVC = What storage does my application need?

StorageClass = How should that storage be provisioned?
```