# Kubernetes Services – Quick Revision

This document contains quick revision notes for **Kubernetes Services**, including why Services are required, Service discovery, load balancing, port forwarding, ClusterIP, NodePort, LoadBalancer, multiple ports, and useful troubleshooting commands.

---

# 1. What is a Kubernetes Service?

A **Kubernetes Service** is an abstraction that provides a stable network endpoint to access a group of Pods.

Pods are temporary and can be created, deleted, or recreated by Kubernetes.

A Service provides a stable way for other applications or users to communicate with those Pods.

### Simple architecture

```text
                    Kubernetes Service
                    nginx-service
                         │
              Stable IP + DNS Name
                         │
          ┌──────────────┼──────────────┐
          ↓              ↓              ↓
       Pod 1           Pod 2           Pod 3
      nginx            nginx           nginx
```

---

# 2. Why Do We Need Services?

One of the biggest problems with directly accessing Pods is that **Pod IP addresses are not permanent**.

For example:

```text
Pod 1 → 10.244.0.10
Pod 2 → 10.244.0.11
Pod 3 → 10.244.0.12
```

If Pod 2 is deleted and Kubernetes creates a replacement:

```text
Old Pod 2 → 10.244.0.11 ❌

New Pod 2 → 10.244.0.25 ✅
```

The IP address changed.

If another application was directly communicating with `10.244.0.11`, communication would break.

### Service solves this problem

```text
Application
     │
     ↓
Service
nginx-service
10.x.x.x
     │
     ├── Pod 1
     ├── Pod 2
     └── Pod 3
```

The **Service IP remains stable**, while Pods behind the Service can change.

---

# 3. What Does a Service Provide?

A Kubernetes Service provides:

### 1. Stable Networking

Provides a stable IP address and DNS name.

### 2. Service Discovery

Applications can find other applications using the Service name.

Example:

```text
http://nginx-service
```

instead of:

```text
http://10.244.0.15
```

### 3. Load Balancing

Traffic sent to a Service can be distributed across the Pods selected by that Service.

```text
                Service
                   │
        ┌──────────┼──────────┐
        ↓          ↓          ↓
      Pod 1      Pod 2      Pod 3
       33%        33%        34%
```

### 4. Application Availability

If one Pod goes down, the Service can continue sending traffic to healthy Pods.

```text
Service
   │
   ├── Pod 1 ✅
   ├── Pod 2 ❌
   └── Pod 3 ✅
```

The Service does not send traffic to the unavailable Pod if it is no longer an active endpoint.

---

# 4. Service IP

When a Service is created, Kubernetes assigns it a **stable virtual IP** for its lifetime.

Example:

```text
Service Name: nginx-service
ClusterIP:    10.96.125.20
```

Pods can change their IP addresses, but the Service provides a stable endpoint.

### Important

The Service IP is **not the same as a Pod IP**.

```text
Pod IP       → Temporary
Service IP   → Stable for the Service lifetime
```

---

# 5. Service Ports

A Service can expose an application through a specific port.

Example:

```yaml
ports:
  - port: 8080
    targetPort: 80
```

Here:

```text
Service Port       → 8080
Pod Container Port → 80
```

Traffic flow:

```text
Client
  │
  │ :8080
  ↓
Service
  │
  │ :80
  ↓
Pod
```

---

# 6. `port` vs `targetPort`

This is an important interview question.

### `port`

The port exposed by the **Service**.

### `targetPort`

The port on the **Pod/container** where the application is listening.

Example:

```yaml
ports:
  - port: 8080
    targetPort: 80
```

Means:

```text
Service :8080
     ↓
Pod :80
```

---

# 7. ClusterIP Service

**ClusterIP** is the default Kubernetes Service type.

Example:

```yaml
apiVersion: v1
kind: Service

metadata:
  name: nginx-service

spec:
  type: ClusterIP

  selector:
    app: nginx-demo

  ports:
    - port: 8080
      targetPort: 80
```

You can also omit:

```yaml
type: ClusterIP
```

because ClusterIP is the default Service type.

---

## ClusterIP Architecture

```text
                Kubernetes Cluster

Application ───────→ nginx-service
                       ClusterIP
                          │
                 ┌────────┼────────┐
                 ↓        ↓        ↓
               Pod 1    Pod 2    Pod 3
```

### Important

A ClusterIP Service is normally accessible **only from inside the Kubernetes cluster**.

It is commonly used for internal communication such as:

```text
Frontend → Backend Service
Backend  → Database Service
Backend  → Redis Service
```

---

# 8. Service Discovery

Kubernetes provides DNS-based Service discovery.

Suppose we have:

```text
Service Name:
nginx-service
```

Applications inside the cluster can generally access it using:

```text
http://nginx-service
```

Instead of using the Service IP directly.

This is one of the major benefits of Services.

```text
Application
     │
     ↓
nginx-service
     │
     ↓
ClusterIP
     │
     ↓
Pods
```

---

# 9. Multiple Ports in a Service

A single Service can expose **multiple ports**.

When using multiple ports, each port should have a unique name.

Example:

```yaml
ports:

  - name: api-port
    port: 8080
    targetPort: 80080

  - name: ui-port
    port: 8083
    targetPort: 80
```

Traffic flow:

```text
API:

Client
  ↓
Service :8080
  ↓
Pod :80080


UI:

Client
  ↓
Service :8083
  ↓
Pod :80
```

### Important

The port names should normally follow Kubernetes naming conventions, so prefer:

```yaml
api-port
ui-port
```

rather than:

```yaml
Api-Port
Ui-Port
```

Also, double-check your example:

```yaml
targetPort: 80080
```

Port `80080` is outside the valid TCP/UDP port range. If this was intended to be `8008`, `8080`, or another port, use the actual port your application listens on.

---

# 10. Port Forwarding

Port forwarding allows us to temporarily forward a local port on our laptop to a Pod or Service inside Kubernetes.

Example:

```bash
kubectl port-forward service/nginx-service 8080:80
```

This means:

```text
Your Laptop
localhost:8080
      │
      │ Port Forward
      ↓
Kubernetes Service
      │
      ↓
Pod :80
```

You can then access:

```text
http://localhost:8080
```

---

# 11. Port Forwarding in Production

Port forwarding is mainly useful for:

* Debugging
* Troubleshooting
* Development
* Temporary testing
* Accessing an internal application

It should **not normally be used as the production method for exposing applications to users**.

Example:

```bash
kubectl port-forward service/backend-service 8080:8080
```

This can temporarily allow you to test the backend from your laptop.

### Production exposure

For production traffic, use appropriate Kubernetes networking such as:

```text
ClusterIP
NodePort
LoadBalancer
Ingress / Gateway
```

depending on the architecture and environment.

### Easy interview answer

> Port forwarding is mainly a temporary debugging and troubleshooting mechanism, not a production application-exposure mechanism.

---

# 12. Port Forward a Service to Your Laptop

Command:

```bash
kubectl port-forward service/nginx-service 8080:80
```

Generic syntax:

```bash
kubectl port-forward service/<service-name> <local-port>:<service-port>
```

Example:

```bash
kubectl port-forward service/nginx-service 9090:8080
```

Traffic:

```text
localhost:9090
      ↓
Service :8080
      ↓
Pod
```

---

# 13. Useful Service Commands

### List Services

```bash
kubectl get services
```

or:

```bash
kubectl get svc
```

### Get detailed Service information

```bash
kubectl describe service/nginx-service
```

### Generic

```bash
kubectl describe service/<service-name>
```

### Check Service endpoints

```bash
kubectl get endpoints
```

or:

```bash
kubectl get endpoints nginx-service
```

### Check Service using EndpointSlice

Modern Kubernetes clusters also use EndpointSlices:

```bash
kubectl get endpointslices
```

---

# 14. Endpoints

Endpoints show the Pod IP addresses currently associated with a Service.

Example:

```text
NAME            ENDPOINTS
nginx-service   10.244.0.10:80,
                10.244.0.11:80,
                10.244.0.12:80
```

Architecture:

```text
Service
   │
   ├── Endpoint → Pod 1
   ├── Endpoint → Pod 2
   └── Endpoint → Pod 3
```

If a Pod becomes unavailable, its endpoint can be removed from the active backend set.

---

# 15. NodePort Service

A **NodePort** Service exposes an application on a port of each Kubernetes Node.

Example:

```yaml
apiVersion: v1
kind: Service

metadata:
  name: nginx-nodeport

spec:
  type: NodePort

  selector:
    app: nginx-demo

  ports:
    - port: 80
      targetPort: 80
      nodePort: 30080
```

Traffic flow:

```text
External Client
      │
      ↓
Node IP :30080
      │
      ↓
NodePort Service
      │
      ↓
Pod :80
```

### NodePort range

The default NodePort range is:

```text
30000 - 32767
```

unless the cluster configuration has been changed.

---

# 16. NodePort vs ClusterIP

| Feature                 | ClusterIP  | NodePort |
| ----------------------- | ---------- | -------- |
| Default Service type    | ✅          | ❌        |
| Internal cluster access | ✅          | ✅        |
| External access         | Normally ❌ | ✅        |
| Stable Service IP       | ✅          | ✅        |
| Exposes Node port       | ❌          | ✅        |

### Easy memory trick

```text
ClusterIP
→ Inside cluster

NodePort
→ Node IP + Port
```

---

# 17. LoadBalancer Service

A **LoadBalancer** Service is commonly used when an external load balancer should expose the application.

Example:

```yaml
apiVersion: v1
kind: Service

metadata:
  name: nginx-loadbalancer

spec:
  type: LoadBalancer

  selector:
    app: nginx-demo

  ports:
    - port: 80
      targetPort: 80
```

In a cloud environment, the cloud provider can provision an external load balancer.

Architecture:

```text
                  Internet
                     │
                     ↓
            External Load Balancer
                     │
                     ↓
              Kubernetes Service
                     │
          ┌──────────┼──────────┐
          ↓          ↓          ↓
        Pod 1      Pod 2      Pod 3
```

You can check the Service with:

```bash
kubectl get service nginx-loadbalancer
```

You may see something similar to:

```text
NAME                TYPE           CLUSTER-IP      EXTERNAL-IP
nginx-loadbalancer  LoadBalancer   10.96.120.10    34.x.x.x
```

The exact behavior depends on the Kubernetes environment and cloud provider.

---

# 18. Service Types – Quick Revision

The three Service types covered here are:

```text
ClusterIP
   ↓
Internal cluster access


NodePort
   ↓
Expose through Node IP + Port


LoadBalancer
   ↓
External Load Balancer
```

### Quick comparison

| Service      | Main Purpose                            |
| ------------ | --------------------------------------- |
| ClusterIP    | Internal application communication      |
| NodePort     | Basic external access through Node port |
| LoadBalancer | External access through a load balancer |

---

# 19. Complete Example – Deployment + Service

### Deployment

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: nginx-deployment-demo

spec:
  replicas: 3

  selector:
    matchLabels:
      app: nginx-demo

  template:
    metadata:
      labels:
        app: nginx-demo

    spec:
      containers:
        - name: nginx
          image: nginx:latest
          ports:
            - containerPort: 80
```

### ClusterIP Service

```yaml
apiVersion: v1
kind: Service

metadata:
  name: nginx-service

spec:
  type: ClusterIP

  selector:
    app: nginx-demo

  ports:
    - port: 80
      targetPort: 80
```

Notice that:

```yaml
selector:
  app: nginx-demo
```

matches the Deployment's Pod label:

```yaml
labels:
  app: nginx-demo
```

This is how the Service knows which Pods should receive traffic.

---

# 20. Important Concept – Service Selector

A Service normally selects Pods using labels.

Example:

```yaml
selector:
  app: nginx-demo
```

Pods:

```yaml
labels:
  app: nginx-demo
```

Therefore:

```text
Service
selector: app=nginx-demo
          │
          ├── Pod 1 → app=nginx-demo ✅
          ├── Pod 2 → app=nginx-demo ✅
          ├── Pod 3 → app=nginx-demo ✅
          └── Pod 4 → app=apache-demo ❌
```

Only matching Pods become Service backends.

---

# 21. Troubleshooting a Service

If your Service isn't forwarding traffic correctly, check:

### Step 1 – Check Service

```bash
kubectl get svc
```

### Step 2 – Describe Service

```bash
kubectl describe service/nginx-service
```

### Step 3 – Check endpoints

```bash
kubectl get endpoints nginx-service
```

### Step 4 – Check Pods

```bash
kubectl get pods -o wide
```

### Step 5 – Check Pod labels

```bash
kubectl get pods --show-labels
```

### Step 6 – Check Service selector

```bash
kubectl describe service/nginx-service
```

Make sure:

```text
Service selector
        ↓
matches
        ↓
Pod labels
```

---

# 22. Interview Questions

### Q: Why do we need a Service?

Because Pod IP addresses are dynamic and can change when Pods are recreated. A Service provides a stable network endpoint.

### Q: What are the main benefits of a Service?

```text
Stable networking
Service discovery
Load balancing
Application availability
```

### Q: What is the default Service type?

```text
ClusterIP
```

### Q: What is the difference between `port` and `targetPort`?

```text
port       → Service port
targetPort → Pod/application port
```

### Q: Can a Service have multiple ports?

Yes.

Example:

```yaml
ports:
  - name: api-port
    port: 8080
    targetPort: 8080

  - name: ui-port
    port: 8083
    targetPort: 80
```

### Q: What is port forwarding?

A temporary connection from a local machine to a Pod or Service inside Kubernetes, primarily useful for development and debugging.

### Q: Is port forwarding recommended for production traffic?

No. It is primarily a temporary debugging/testing mechanism.

### Q: What does NodePort do?

It exposes a Service through a port on each Kubernetes Node.

### Q: What does LoadBalancer do?

It exposes a Service through an external load-balancing mechanism, typically provided by the cloud/platform environment.

---

# 23. Quick Commands Cheat Sheet

```bash
# List Services
kubectl get svc

# Describe Service
kubectl describe service/<service-name>

# Get endpoints
kubectl get endpoints

# Get specific endpoints
kubectl get endpoints <service-name>

# Get EndpointSlices
kubectl get endpointslices

# Port forward Service to local machine
kubectl port-forward service/<service-name> 8080:80

# Apply Service YAML
kubectl apply -f service.yaml

# Delete Service
kubectl delete service <service-name>
```

---

# 24. Final Memory Map

```text
                         Kubernetes Service
                                │
              ┌─────────────────┼─────────────────┐
              │                 │                 │
         Stable IP        Service Discovery   Load Balancing
              │                 │                 │
              └─────────────────┼─────────────────┘
                                │
                         Selects Pods
                                │
                   ┌────────────┼────────────┐
                   ↓            ↓            ↓
                 Pod 1        Pod 2        Pod 3
```

### Service Types

```text
ClusterIP
   │
   └── Internal cluster communication


NodePort
   │
   └── Node IP + Port


LoadBalancer
   │
   └── External Load Balancer
```

### Most Important Things to Remember

```text
Pod IP       → Can change
Service IP   → Stable

Service      → Stable endpoint for Pods

port         → Service port
targetPort   → Pod/application port

ClusterIP    → Internal access
NodePort     → Node IP + Port
LoadBalancer → External Load Balancer

Port Forward → Temporary debugging/testing
```

> **Interview one-liner:**
> A Kubernetes Service provides a stable network endpoint and DNS-based service discovery for a dynamic set of Pods, while also providing load balancing and improving application availability.
