# Kubernetes

---

## Table of Contents
1. [Core Concepts (Q1–Q20)](#1-core-concepts)
2. [Workloads — Pods, Deployments, Services (Q21–Q40)](#2-workloads--pods-deployments-services)
3. [Storage & Configuration (Q41–Q55)](#3-storage--configuration)
4. [Networking (Q56–Q67)](#4-networking)
5. [Scheduling & Resource Management (Q68–Q78)](#5-scheduling--resource-management)
6. [Security (Q79–Q88)](#6-security)
7. [kubectl Commands (Q89–Q95)](#7-kubectl-commands)
8. [Advanced & Real-World Scenarios (Q96–Q110)](#8-advanced--real-world-scenarios)

---

## 1. Core Concepts

**Q1. What is Kubernetes?**
> Kubernetes (K8s) is an open-source container orchestration platform. It automates:
> - **Deployment** of containers
> - **Scaling** up and down
> - **Self-healing** (restart failed containers)
> - **Load balancing** across containers
> - **Rolling updates** with zero downtime
> Think of it as an operating system for your cluster of machines.

---

**Q2. Why do we need Kubernetes if we have Docker?**
> Docker runs containers on ONE machine. Kubernetes manages containers across MANY machines.
> | Problem | Docker alone | Kubernetes |
> |---|---|---|
> | Container crashes | Manual restart | Auto-restarts |
> | High traffic | Manual scaling | Auto-scaling |
> | Machine fails | App goes down | Reschedules on healthy node |
> | Deploy update | Downtime | Zero-downtime rolling update |
> | Load balancing | Manual setup | Built-in |

---

**Q3. What is a Kubernetes Cluster?**
> A cluster is a set of machines (nodes) that run containerized applications managed by Kubernetes.
> - **Control Plane (Master):** Manages the cluster — scheduling, API, state
> - **Worker Nodes:** Actually run the application containers
> A production cluster typically has 3 control plane nodes (HA) and multiple worker nodes.

---

**Q4. What are the components of the Control Plane?**
> - **kube-apiserver:** Front door of Kubernetes. All communication goes through it. REST API.
> - **etcd:** Key-value database that stores ALL cluster state (the brain's memory)
> - **kube-scheduler:** Decides which node to run a new Pod on
> - **kube-controller-manager:** Runs controllers that maintain desired state (ReplicaSet, Node, Job controllers)
> - **cloud-controller-manager:** Integrates with cloud providers (AWS, GCP, Azure)

---

**Q5. What are the components of a Worker Node?**
> - **kubelet:** Agent on every node. Gets Pod specs from API server and ensures containers are running.
> - **kube-proxy:** Manages network rules on the node. Implements Service load balancing.
> - **Container Runtime:** Runs containers (containerd, CRI-O). Docker was removed in K8s 1.24+.

---

**Q6. What is a Pod?**
> A Pod is the smallest deployable unit in Kubernetes. It's a wrapper around one or more containers that:
> - Share the same network (same IP address)
> - Share the same storage volumes
> - Are always scheduled together on the same node
> Usually 1 container per Pod. Multi-container Pods use sidecar pattern.

---

**Q7. What is etcd and why is it important?**
> etcd is a distributed key-value store that stores ALL Kubernetes cluster data:
> - All resource definitions (Pods, Services, Deployments)
> - Cluster state and configuration
> - Secrets
> If etcd is lost, the cluster loses all state. That's why backing up etcd is critical in production.

---

**Q8. What is `kubectl`?**
> `kubectl` is the command-line tool to interact with a Kubernetes cluster. It talks to the kube-apiserver.
```bash
kubectl get pods                    # list pods
kubectl apply -f deployment.yaml    # create/update resources
kubectl delete pod mypod            # delete a pod
kubectl logs mypod                  # view pod logs
kubectl exec -it mypod -- bash      # open shell in pod
```

---

**Q9. What is a Namespace in Kubernetes?**
> Namespaces provide logical isolation within a cluster. Like folders — separate environments in one cluster.
```bash
kubectl get pods -n production      # pods in production namespace
kubectl get pods --all-namespaces   # pods in all namespaces
```
> Default namespaces:
> - `default` — where resources go if you don't specify
> - `kube-system` — Kubernetes system components
> - `kube-public` — publicly readable resources
> - `kube-node-lease` — node heartbeat objects

---

**Q10. What is the difference between Kubernetes and OpenShift?**
> OpenShift is Red Hat's enterprise Kubernetes platform with extra features:
> - Built-in CI/CD (Tekton, Jenkins)
> - Stricter security (no root containers by default)
> - Developer-friendly web console
> - Image builds on cluster (Source-to-Image)
> OpenShift runs ON TOP of Kubernetes — it's Kubernetes + enterprise features.

---

**Q11. What is a ReplicaSet?**
> A ReplicaSet ensures a specified number of Pod replicas are always running. If a Pod dies, ReplicaSet creates a new one.
```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: myapp-rs
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: myapp
        image: myapp:1.0
```
> In practice, you rarely create ReplicaSets directly — Deployments manage them.

---

**Q12. What is the difference between ReplicaSet and ReplicationController?**
> ReplicationController is the old version of ReplicaSet. ReplicaSet supports:
> - Set-based label selectors (`In`, `NotIn`, `Exists`)
> - Better flexibility
> Always use ReplicaSet (or Deployment) — ReplicationController is deprecated.

---

**Q13. What is a DaemonSet?**
> A DaemonSet ensures ONE Pod runs on EVERY node (or specific nodes). Used for:
> - Log collectors (Fluentd, Filebeat) — one per node
> - Monitoring agents (Prometheus node exporter) — one per node
> - Network plugins (Calico, Weave) — one per node
> When a new node joins the cluster, DaemonSet automatically adds a Pod to it.

---

**Q14. What is a StatefulSet?**
> StatefulSet manages stateful applications (databases, Kafka, Zookeeper). Unlike Deployments:
> - Pods get stable, predictable names: `mysql-0`, `mysql-1`, `mysql-2`
> - Pods start and stop in order
> - Each Pod gets its own persistent volume
> - Stable network identity (DNS: `mysql-0.mysql-service.namespace.svc.cluster.local`)
> Use for: MySQL, PostgreSQL, MongoDB, Kafka, Elasticsearch, Redis Cluster.

---

**Q15. What is a Job and CronJob in Kubernetes?**
> - **Job:** Runs a Pod to completion (not forever). Useful for batch processing, database migrations.
>   ```yaml
>   kind: Job
>   spec:
>     completions: 1      # run 1 Pod to completion
>     parallelism: 1      # run 1 at a time
>   ```
> - **CronJob:** Runs a Job on a schedule (like Linux cron).
>   ```yaml
>   kind: CronJob
>   spec:
>     schedule: "0 2 * * *"   # every day at 2 AM
>   ```

---

**Q16. What is a ConfigMap?**
> ConfigMap stores non-sensitive configuration data as key-value pairs. Decouples config from container images.
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  APP_ENV: production
  LOG_LEVEL: info
  DB_PORT: "5432"
  config.yaml: |
    server:
      port: 8080
      timeout: 30s
```

---

**Q17. What is a Secret in Kubernetes?**
> Secret stores sensitive data (passwords, tokens, keys) encoded in base64.
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: db-secret
type: Opaque
data:
  username: cG9zdGdyZXM=     # base64 of "postgres"
  password: c2VjcmV0MTIz     # base64 of "secret123"
```
> ⚠️ base64 is NOT encryption — Secrets are not secure by default! Use:
> - etcd encryption at rest
> - External secrets (AWS Secrets Manager, HashiCorp Vault)
> - Sealed Secrets

---

**Q18. What is a Service in Kubernetes?**
> A Service provides a stable network endpoint (IP + DNS) to access a group of Pods. Since Pods are ephemeral (die and get new IPs), Services provide a consistent access point.
> Types: ClusterIP, NodePort, LoadBalancer, ExternalName.

---

**Q19. What is a Label and Selector in Kubernetes?**
> **Labels** are key-value pairs attached to resources. **Selectors** filter resources by labels.
```yaml
# Pod with labels
metadata:
  labels:
    app: myapp
    env: production
    version: v2

# Service selector - sends traffic to pods with these labels
selector:
  app: myapp
  env: production
```
> This is how Services find their Pods, and how Deployments manage ReplicaSets.

---

**Q20. What is an Annotation in Kubernetes?**
> Annotations are key-value pairs for non-identifying metadata — used by tools and systems, not for selection.
```yaml
metadata:
  annotations:
    deployment.kubernetes.io/revision: "3"
    kubectl.kubernetes.io/last-applied-configuration: "..."
    prometheus.io/scrape: "true"
    prometheus.io/port: "8080"
```
> Labels = for selecting/grouping. Annotations = for extra information/tooling.

---

## 2. Workloads — Pods, Deployments, Services

**Q21. Write a basic Pod YAML.**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app: myapp
spec:
  containers:
  - name: myapp
    image: nginx:1.25
    ports:
    - containerPort: 80
    resources:
      requests:
        memory: "64Mi"
        cpu: "250m"
      limits:
        memory: "128Mi"
        cpu: "500m"
    env:
    - name: ENV
      value: "production"
```

---

**Q22. What is a Deployment and write its YAML.**
> A Deployment manages ReplicaSets and provides declarative updates. It's the standard way to run stateless apps.
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-deployment
  namespace: production
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1         # max extra pods during update
      maxUnavailable: 0   # never have less than desired pods
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: myapp
        image: myrepo/myapp:1.0
        ports:
        - containerPort: 8080
        readinessProbe:
          httpGet:
            path: /health
            port: 8080
          initialDelaySeconds: 10
          periodSeconds: 5
        livenessProbe:
          httpGet:
            path: /health
            port: 8080
          initialDelaySeconds: 30
          periodSeconds: 10
        resources:
          requests:
            memory: "128Mi"
            cpu: "100m"
          limits:
            memory: "256Mi"
            cpu: "500m"
```

---

**Q23. What are the types of Kubernetes Services?**
> - **ClusterIP (default):** Internal-only IP. Only accessible within the cluster.
> - **NodePort:** Opens a port on every node (30000–32767). Accessible from outside via `NodeIP:NodePort`.
> - **LoadBalancer:** Creates a cloud load balancer (AWS ELB, GCP LB). Gets an external IP. Most common for production.
> - **ExternalName:** Maps service to an external DNS name. No proxying.

---

**Q24. Write a Service YAML for each type.**
```yaml
# ClusterIP - internal access only
apiVersion: v1
kind: Service
metadata:
  name: myapp-clusterip
spec:
  type: ClusterIP
  selector:
    app: myapp
  ports:
  - port: 80          # service port
    targetPort: 8080  # container port

---
# NodePort - external access via node IP
apiVersion: v1
kind: Service
metadata:
  name: myapp-nodeport
spec:
  type: NodePort
  selector:
    app: myapp
  ports:
  - port: 80
    targetPort: 8080
    nodePort: 30080   # optional, auto-assigned if omitted

---
# LoadBalancer - cloud load balancer
apiVersion: v1
kind: Service
metadata:
  name: myapp-lb
spec:
  type: LoadBalancer
  selector:
    app: myapp
  ports:
  - port: 80
    targetPort: 8080
```

---

**Q25. What is a Headless Service?**
> A Headless Service has `clusterIP: None`. Instead of a single IP, DNS returns the IPs of all Pods directly. Used with StatefulSets for direct Pod-to-Pod communication.
```yaml
spec:
  clusterIP: None
  selector:
    app: mysql
```
> DNS: `mysql-0.mysql-headless.namespace.svc.cluster.local` → Pod IP directly.

---

**Q26. What is the difference between Rolling Update and Recreate deployment strategy?**
> - **RollingUpdate (default):** Gradually replaces old Pods with new ones. Zero downtime. Configurable with `maxSurge` and `maxUnavailable`.
> - **Recreate:** Kills ALL old Pods first, then creates new ones. Has downtime. Use when old and new versions can't run simultaneously (DB schema changes).

---

**Q27. How do you rollback a Deployment?**
```bash
# Check rollout history
kubectl rollout history deployment/myapp

# Rollback to previous version
kubectl rollout undo deployment/myapp

# Rollback to specific revision
kubectl rollout undo deployment/myapp --to-revision=2

# Check rollout status
kubectl rollout status deployment/myapp
```

---

**Q28. What is a Liveness Probe?**
> Liveness probe checks if a container is alive. If it fails, Kubernetes RESTARTS the container.
```yaml
livenessProbe:
  httpGet:
    path: /health
    port: 8080
  initialDelaySeconds: 30   # wait 30s before first check
  periodSeconds: 10          # check every 10s
  failureThreshold: 3        # restart after 3 consecutive failures
```
> Use for: Detect deadlocks, infinite loops, crashes that don't exit.

---

**Q29. What is a Readiness Probe?**
> Readiness probe checks if a container is ready to receive traffic. If it fails, Kubernetes REMOVES the Pod from Service endpoints (stops sending traffic). Does NOT restart the container.
```yaml
readinessProbe:
  httpGet:
    path: /ready
    port: 8080
  initialDelaySeconds: 10
  periodSeconds: 5
  failureThreshold: 3
```
> Use for: App needs time to warm up, load config, connect to DB before serving traffic.

---

**Q30. What is a Startup Probe?**
> Startup probe is for slow-starting applications. While startup probe is active, liveness and readiness probes are disabled. Only after startup probe succeeds do others kick in.
```yaml
startupProbe:
  httpGet:
    path: /health
    port: 8080
  failureThreshold: 30   # up to 30 * 10s = 5 minutes to start
  periodSeconds: 10
```
> Use for: Legacy apps, JVM apps with long startup times.

---

**Q31. What is the difference between Liveness, Readiness, and Startup probes?**
> | Probe | On Failure | Use Case |
> |---|---|---|
> | Liveness | Restarts container | Detect deadlock/crash |
> | Readiness | Removes from Service | App not ready for traffic |
> | Startup | Keeps checking | Slow starting apps |

---

**Q32. What is an Init Container?**
> Init containers run BEFORE the main container starts. They run to completion, then the main container starts.
```yaml
spec:
  initContainers:
  - name: wait-for-db
    image: busybox
    command: ['sh', '-c', 'until nc -z db-service 5432; do sleep 2; done']
  - name: run-migrations
    image: myapp:latest
    command: ['python', 'manage.py', 'migrate']
  containers:
  - name: myapp
    image: myapp:latest
```
> Use for: Wait for dependencies, run DB migrations, set up config files.

---

**Q33. What is a Sidecar container in Kubernetes?**
> A sidecar is an additional container in the same Pod that supports the main container.
```yaml
spec:
  containers:
  - name: myapp          # main container
    image: myapp:1.0
  - name: log-shipper    # sidecar
    image: fluentd:1.0
    volumeMounts:
    - name: logs
      mountPath: /var/log/app
```
> Common sidecars: Log shippers, Envoy proxy (Istio), config reloaders, metrics exporters.

---

**Q34. What is a Horizontal Pod Autoscaler (HPA)?**
> HPA automatically scales the number of Pod replicas based on metrics (CPU, memory, custom metrics).
```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: myapp-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: myapp
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70  # scale up if CPU > 70%
```

---

**Q35. What is a Vertical Pod Autoscaler (VPA)?**
> VPA automatically adjusts the CPU and memory **requests/limits** of containers based on actual usage.
> - HPA = scales number of pods (out/in)
> - VPA = scales resources of existing pods (up/down)
> VPA and HPA should not be used together on same metric (conflict).

---

**Q36. What is a PodDisruptionBudget (PDB)?**
> PDB limits how many Pods of an application can be down simultaneously during voluntary disruptions (node drain, upgrades).
```yaml
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: myapp-pdb
spec:
  minAvailable: 2      # always keep at least 2 pods running
  # OR
  maxUnavailable: 1    # at most 1 pod can be unavailable
  selector:
    matchLabels:
      app: myapp
```

---

**Q37. What is a Kubernetes Ingress?**
> Ingress manages external HTTP/HTTPS traffic to Services inside the cluster. It's like a smart router:
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: myapp-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx
  rules:
  - host: myapp.example.com
    http:
      paths:
      - path: /api
        pathType: Prefix
        backend:
          service:
            name: api-service
            port:
              number: 80
      - path: /
        pathType: Prefix
        backend:
          service:
            name: frontend-service
            port:
              number: 80
  tls:
  - hosts:
    - myapp.example.com
    secretName: myapp-tls-secret
```

---

**Q38. What is an Ingress Controller?**
> Ingress Controller is the actual software that reads Ingress rules and configures the load balancer. Popular options:
> - **NGINX Ingress Controller** — most popular, open source
> - **Traefik** — cloud-native, auto-discovers services
> - **AWS ALB Ingress Controller** — uses AWS Application Load Balancer
> - **HAProxy** — high performance
> Ingress resource = rules. Ingress Controller = the engine that applies the rules.

---

**Q39. What is a Kubernetes Operator?**
> An Operator is a custom controller that extends Kubernetes to manage complex stateful apps. It encodes human operational knowledge into software.
> Examples:
> - **Prometheus Operator** — manages Prometheus, Alertmanager deployment
> - **cert-manager** — manages SSL certificates
> - **MySQL Operator** — manages MySQL clusters with backups, failover
> - **Strimzi** — manages Kafka on Kubernetes

---

**Q40. What is Helm?**
> Helm is the package manager for Kubernetes. It packages K8s manifests into **Charts** (like apt/yum packages).
```bash
helm install myapp ./mychart               # install chart
helm upgrade myapp ./mychart               # upgrade release
helm rollback myapp 1                      # rollback to revision 1
helm list                                  # list installed releases
helm uninstall myapp                       # remove release

# Install from public repo
helm repo add bitnami https://charts.bitnami.com/bitnami
helm install my-postgres bitnami/postgresql
```

---

## 3. Storage & Configuration

**Q41. What is a PersistentVolume (PV)?**
> A PersistentVolume is a piece of storage in the cluster provisioned by an admin (or dynamically). It's a cluster-level resource, independent of any Pod.
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: my-pv
spec:
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: standard
  hostPath:
    path: /data/myapp   # for local dev only
```

---

**Q42. What is a PersistentVolumeClaim (PVC)?**
> A PVC is a request for storage by a user. Kubernetes finds a matching PV and binds them together.
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: myapp-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
  storageClassName: standard
```
```yaml
# Use in Pod
volumes:
- name: mydata
  persistentVolumeClaim:
    claimName: myapp-pvc
```

---

**Q43. What are PV Access Modes?**
> - **ReadWriteOnce (RWO):** Mounted read-write by ONE node. Most common (EBS, local disk).
> - **ReadOnlyMany (ROX):** Mounted read-only by MANY nodes.
> - **ReadWriteMany (RWX):** Mounted read-write by MANY nodes simultaneously. (EFS, NFS, CephFS)
> - **ReadWriteOncePod (RWOP):** Mounted read-write by ONE pod only (K8s 1.22+).

---

**Q44. What is a StorageClass?**
> StorageClass enables dynamic provisioning of PersistentVolumes. Instead of manually creating PVs, Kubernetes automatically creates them when a PVC is made.
```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: fast-ssd
provisioner: kubernetes.io/aws-ebs
parameters:
  type: gp3
  iops: "3000"
  throughput: "125"
reclaimPolicy: Delete
allowVolumeExpansion: true
```

---

**Q45. What is the PV Reclaim Policy?**
> What happens to the PV when the PVC is deleted:
> - **Retain:** PV and data kept. Must manually reclaim. (Safe for production data)
> - **Delete:** PV and underlying storage deleted automatically. (Common for dynamic provisioning)
> - **Recycle:** Basic scrub and make available again. (Deprecated)

---

**Q46. How do you use a ConfigMap in a Pod?**
```yaml
# Method 1: As environment variables
envFrom:
- configMapRef:
    name: app-config

# Method 2: Specific key as env var
env:
- name: LOG_LEVEL
  valueFrom:
    configMapKeyRef:
      name: app-config
      key: LOG_LEVEL

# Method 3: Mount as files (volume)
volumes:
- name: config-volume
  configMap:
    name: app-config
volumeMounts:
- name: config-volume
  mountPath: /etc/config
```

---

**Q47. How do you use a Secret in a Pod?**
```yaml
# As environment variable
env:
- name: DB_PASSWORD
  valueFrom:
    secretKeyRef:
      name: db-secret
      key: password

# Mount as file (more secure - not exposed in env)
volumes:
- name: secret-volume
  secret:
    secretName: db-secret
volumeMounts:
- name: secret-volume
  mountPath: /etc/secrets
  readOnly: true
```

---

**Q48. What is the difference between ConfigMap and Secret?**
> | Feature | ConfigMap | Secret |
> |---|---|---|
> | Data type | Non-sensitive config | Sensitive data |
> | Storage | Plain text in etcd | Base64 encoded in etcd |
> | Size limit | 1MB | 1MB |
> | Use case | App config, feature flags | Passwords, tokens, certs |

---

**Q49. What is an EmptyDir volume?**
> EmptyDir is a temporary volume created when a Pod is assigned to a node. Deleted when Pod is removed. Shared between all containers in a Pod.
```yaml
volumes:
- name: shared-data
  emptyDir: {}
  # or in-memory:
  # emptyDir:
  #   medium: Memory
  #   sizeLimit: 256Mi
```
> Use for: Sharing files between containers in same Pod, cache, scratch space.

---

**Q50. What is a Projected Volume?**
> Projects multiple volume sources into a single directory:
```yaml
volumes:
- name: all-in-one
  projected:
    sources:
    - secret:
        name: mysecret
    - configMap:
        name: myconfig
    - serviceAccountToken:
        path: token
        expirationSeconds: 3600
```

---

**Q51. What is a ResourceQuota?**
> ResourceQuota limits the total resources a namespace can use:
```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: production-quota
  namespace: production
spec:
  hard:
    pods: "50"
    requests.cpu: "10"
    requests.memory: 20Gi
    limits.cpu: "20"
    limits.memory: 40Gi
    persistentvolumeclaims: "20"
```

---

**Q52. What is a LimitRange?**
> LimitRange sets default and max/min resource limits for containers in a namespace:
```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: default-limits
spec:
  limits:
  - type: Container
    default:          # default limits if not specified
      cpu: 500m
      memory: 256Mi
    defaultRequest:   # default requests if not specified
      cpu: 100m
      memory: 128Mi
    max:              # maximum allowed
      cpu: "2"
      memory: 1Gi
    min:              # minimum required
      cpu: 50m
      memory: 64Mi
```

---

**Q53. What is External Secrets Operator?**
> External Secrets Operator syncs secrets from external sources (AWS Secrets Manager, HashiCorp Vault, GCP Secret Manager) into Kubernetes Secrets automatically.
```yaml
apiVersion: external-secrets.io/v1beta1
kind: ExternalSecret
metadata:
  name: db-secret
spec:
  secretStoreRef:
    name: aws-secretsmanager
    kind: SecretStore
  target:
    name: db-secret   # creates this K8s Secret
  data:
  - secretKey: password
    remoteRef:
      key: prod/myapp/db-password
```

---

**Q54. What is Sealed Secrets?**
> Sealed Secrets (by Bitnami) encrypts Kubernetes Secrets so they can be safely stored in Git.
```bash
# Encrypt a secret
kubeseal --format yaml < secret.yaml > sealed-secret.yaml
# Now sealed-secret.yaml is safe to commit to Git
# Only the controller in the cluster can decrypt it
```

---

**Q55. What is a Volume Snapshot?**
> Volume Snapshots create point-in-time copies of PersistentVolumes (like EBS snapshots for K8s).
```yaml
apiVersion: snapshot.storage.k8s.io/v1
kind: VolumeSnapshot
metadata:
  name: myapp-snapshot
spec:
  volumeSnapshotClassName: csi-aws-vsc
  source:
    persistentVolumeClaimName: myapp-pvc
```

---

## 4. Networking

**Q56. How does Pod networking work in Kubernetes?**
> Every Pod gets a unique IP address. All Pods can communicate with all other Pods without NAT (flat network). This is implemented by CNI plugins.
> Rules:
> - Pod to Pod (same node): Direct via virtual bridge
> - Pod to Pod (different nodes): Via CNI plugin overlay network
> - Pod to Service: Via kube-proxy iptables/IPVS rules

---

**Q57. What is a CNI plugin?**
> CNI (Container Network Interface) plugins implement Pod networking. Popular options:
> - **Calico** — Network policy support, BGP routing, very popular
> - **Flannel** — Simple overlay, easy setup
> - **Weave Net** — Simple, supports encryption
> - **Cilium** — eBPF-based, high performance, advanced security
> - **AWS VPC CNI** — On EKS, Pods get real VPC IPs

---

**Q58. What is a Network Policy?**
> Network Policy controls traffic flow between Pods (like a firewall for Pods). By default all Pods can talk to all Pods.
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-frontend-to-backend
spec:
  podSelector:
    matchLabels:
      app: backend
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: frontend
    ports:
    - protocol: TCP
      port: 8080
  egress:
  - to:
    - podSelector:
        matchLabels:
          app: database
    ports:
    - protocol: TCP
      port: 5432
```

---

**Q59. What is kube-proxy?**
> kube-proxy runs on every node and maintains network rules (iptables or IPVS). It implements the Service abstraction — when you access a Service IP, kube-proxy forwards traffic to one of the Pod IPs.

---

**Q60. What is CoreDNS in Kubernetes?**
> CoreDNS is the DNS server in Kubernetes. It resolves:
> - `my-service` → ClusterIP of service (within same namespace)
> - `my-service.my-namespace` → across namespaces
> - `my-service.my-namespace.svc.cluster.local` → fully qualified
> - External domains → forwarded to upstream DNS

---

**Q61. What is a Service Mesh?**
> A service mesh manages communication between microservices. Adds:
> - **mTLS** between services (encrypted + authenticated)
> - **Traffic management** (canary, A/B testing)
> - **Observability** (automatic metrics, tracing, logs)
> - **Circuit breaking, retries, timeouts**
> Popular options: **Istio**, **Linkerd**, **Consul Connect**

---

**Q62. What is Istio?**
> Istio is the most popular service mesh for Kubernetes. It injects an Envoy sidecar proxy into every Pod automatically. All traffic goes through Envoy — Istio controls it centrally.
> Features:
> - mTLS everywhere (zero trust networking)
> - Traffic splitting (canary deployments)
> - Distributed tracing (Jaeger integration)
> - Circuit breaking
> - Rate limiting

---

**Q63. What is an ExternalName Service?**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: external-db
spec:
  type: ExternalName
  externalName: mydb.rds.amazonaws.com
```
> Maps a K8s service name to an external DNS. Pods use `external-db` — later you can switch to internal DB without code changes.

---

**Q64. What is the difference between Service and Ingress?**
> | Feature | Service | Ingress |
> |---|---|---|
> | Layer | Layer 4 (TCP/UDP) | Layer 7 (HTTP/HTTPS) |
> | Routing | By port | By host, path, headers |
> | External IP | One per service | One for all services |
> | SSL termination | No | Yes |
> | Cost | Multiple LBs = expensive | One LB for all |

---

**Q65. What is Gateway API in Kubernetes?**
> Gateway API is the next-generation successor to Ingress. More powerful, expressive, and role-oriented:
> - **GatewayClass:** Infrastructure (managed by infra team)
> - **Gateway:** Cluster entry point (managed by platform team)
> - **HTTPRoute:** Traffic routing rules (managed by app team)

---

**Q66. What is DNS-based service discovery in K8s?**
> Services register in CoreDNS automatically. Any Pod can reach any service by name:
```bash
# From any pod in same namespace
curl http://myapp-service/api

# From different namespace
curl http://myapp-service.production.svc.cluster.local/api

# Headless service - individual pod
curl http://mysql-0.mysql-headless.default.svc.cluster.local
```

---

**Q67. What is an EndpointSlice?**
> EndpointSlice stores the IP addresses of Pods backing a Service. When Pods are added/removed, EndpointSlice is updated and kube-proxy updates its rules. Replaced the old Endpoints object for better scalability.

---

## 5. Scheduling & Resource Management

**Q68. How does the Kubernetes scheduler work?**
> When a new Pod is created, the scheduler finds the best node:
> 1. **Filtering:** Remove nodes that don't meet requirements (resources, labels, taints)
> 2. **Scoring:** Rank remaining nodes by priority (resource balance, affinity rules)
> 3. **Binding:** Assign Pod to highest-scoring node
> The scheduler talks to kube-apiserver — it never directly contacts nodes.

---

**Q69. What are resource requests and limits?**
> - **Request:** Minimum guaranteed resources. Scheduler uses this to find a node with enough capacity.
> - **Limit:** Maximum resources the container can use. Enforced by the kernel (cgroups).
```yaml
resources:
  requests:
    memory: "128Mi"   # guaranteed 128MB
    cpu: "250m"       # guaranteed 0.25 CPU (250 millicores)
  limits:
    memory: "256Mi"   # max 256MB (killed if exceeded - OOMKill)
    cpu: "500m"       # max 0.5 CPU (throttled if exceeded)
```

---

**Q70. What happens when a container exceeds its memory limit?**
> The container is **OOMKilled** (Out Of Memory Killed) by the Linux kernel. Kubernetes restarts the container. You'll see `OOMKilled` in `kubectl describe pod`.
> Solution: Increase memory limit or fix memory leak in application.

---

**Q71. What are Node Affinity and Pod Affinity?**
> **Node Affinity:** Schedule Pods on specific nodes based on node labels.
```yaml
affinity:
  nodeAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
      nodeSelectorTerms:
      - matchExpressions:
        - key: kubernetes.io/arch
          operator: In
          values: ["amd64"]
    preferredDuringSchedulingIgnoredDuringExecution:
    - weight: 1
      preference:
        matchExpressions:
        - key: node-type
          operator: In
          values: ["gpu"]
```
> **Pod Affinity/Anti-Affinity:** Schedule Pods near or away from other Pods.

---

**Q72. What are Taints and Tolerations?**
> **Taints** mark nodes to repel Pods. **Tolerations** on Pods allow them to schedule on tainted nodes.
```bash
# Taint a node (only GPU workloads allowed)
kubectl taint nodes gpu-node gpu=true:NoSchedule
```
```yaml
# Pod that can tolerate the taint
tolerations:
- key: "gpu"
  operator: "Equal"
  value: "true"
  effect: "NoSchedule"
```
> Taint effects:
> - `NoSchedule` — new Pods won't schedule
> - `PreferNoSchedule` — try to avoid
> - `NoExecute` — evict existing Pods too

---

**Q73. What is a NodeSelector?**
> Simple way to constrain Pods to nodes with specific labels:
```yaml
spec:
  nodeSelector:
    disktype: ssd
    region: us-east-1
```
> Simple but inflexible. Use Node Affinity for more control.

---

**Q74. What is QoS (Quality of Service) in Kubernetes?**
> K8s assigns QoS classes based on resource configuration:
> - **Guaranteed:** `requests == limits` for all containers. Highest priority. Never evicted unless they exceed limits.
> - **Burstable:** `requests < limits`. Medium priority. Evicted if node is under pressure.
> - **BestEffort:** No requests or limits set. Lowest priority. First to be evicted.

---

**Q75. What is a PriorityClass?**
> Assigns priority to Pods. Higher priority Pods preempt (evict) lower priority Pods when resources are scarce.
```yaml
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: high-priority
value: 1000000
globalDefault: false
---
spec:
  priorityClassName: high-priority
```

---

**Q76. What is Cluster Autoscaler?**
> Cluster Autoscaler automatically adds or removes NODES from the cluster:
> - Scales UP: When Pods can't be scheduled due to insufficient resources
> - Scales DOWN: When nodes are underutilized for extended period
> Works with cloud providers (AWS ASG, GCP MIG, Azure VMSS).

---

**Q77. What is the difference between HPA, VPA, and Cluster Autoscaler?**
> - **HPA:** Scales number of Pods based on metrics (CPU, custom)
> - **VPA:** Adjusts CPU/memory of existing Pods
> - **Cluster Autoscaler:** Scales number of Nodes
> Typical combo: HPA scales Pods → Cluster Autoscaler scales Nodes to fit them.

---

**Q78. What is KEDA?**
> KEDA (Kubernetes Event-Driven Autoscaling) scales Pods based on external event sources:
> - Queue length (SQS, RabbitMQ, Kafka)
> - Database record count
> - Prometheus metrics
> - HTTP request rate
> Can scale to ZERO (no queue messages → 0 pods). Great for batch/event-driven workloads.

---

## 6. Security

**Q79. What is RBAC in Kubernetes?**
> RBAC (Role-Based Access Control) controls who can do what in Kubernetes.
> Components:
> - **Role / ClusterRole:** Defines permissions (verbs on resources)
> - **RoleBinding / ClusterRoleBinding:** Assigns a Role to a user/group/ServiceAccount
```yaml
# Role - namespace scoped
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: pod-reader
  namespace: production
rules:
- apiGroups: [""]
  resources: ["pods", "pods/log"]
  verbs: ["get", "list", "watch"]
---
# RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-pods
  namespace: production
subjects:
- kind: User
  name: jane
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

---

**Q80. What is a ServiceAccount?**
> A ServiceAccount is an identity for Pods to authenticate with the Kubernetes API. Every Pod gets a default ServiceAccount. Best practice: Create dedicated ServiceAccounts with minimum permissions.
```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: myapp-sa
  namespace: production
---
# Use in Pod
spec:
  serviceAccountName: myapp-sa
```

---

**Q81. What is a Pod Security Standard (PSS)?**
> PSS defines security policies for Pods. Three levels:
> - **Privileged:** No restrictions (for system/infra components)
> - **Baseline:** Prevents known privilege escalations
> - **Restricted:** Strongly hardened (no root, read-only rootfs, drop all capabilities)
```yaml
# Apply to namespace
apiVersion: v1
kind: Namespace
metadata:
  name: production
  labels:
    pod-security.kubernetes.io/enforce: restricted
    pod-security.kubernetes.io/warn: restricted
```

---

**Q82. What is OPA/Gatekeeper in Kubernetes?**
> OPA (Open Policy Agent) with Gatekeeper lets you write custom admission policies using Rego language.
> Examples of policies you can enforce:
> - All images must come from approved registries
> - All containers must have resource limits
> - No privileged containers
> - All Pods must have labels
> - Containers can't run as root

---

**Q83. What is a MutatingAdmissionWebhook?**
> Admission webhooks intercept API requests before resources are created/updated.
> - **Mutating:** Can MODIFY the request (e.g., inject sidecar, add default labels)
> - **Validating:** Can ACCEPT or REJECT the request (e.g., reject if no resource limits)
> Istio uses mutating webhooks to automatically inject Envoy sidecars.

---

**Q84. What is Falco?**
> Falco is a runtime security tool for Kubernetes. It monitors system calls in real-time and alerts on suspicious behavior:
> - Shell opened inside container
> - Sensitive file read (e.g., /etc/shadow)
> - Privilege escalation attempt
> - Outbound connection to unexpected IP

---

**Q85. What is image scanning in Kubernetes context?**
> Scan container images for CVEs (vulnerabilities) before they run in cluster.
> Tools:
> - **Trivy** — scan in CI/CD pipeline
> - **Clair** — integrated with registries
> - **Snyk** — developer-friendly
> - **Amazon ECR scanning** — automatic scan on push
> Best practice: Block deployment if HIGH/CRITICAL CVEs found (use OPA/Gatekeeper policy).

---

**Q86. What is mTLS and why is it important in Kubernetes?**
> mTLS (mutual TLS) means BOTH client and server verify each other's identity. In K8s:
> - Without mTLS: Pod A trusts any traffic from Pod B (no verification)
> - With mTLS (Istio): Every service has a certificate. Traffic is encrypted AND authenticated.
> Implements zero-trust networking inside the cluster.

---

**Q87. How do you audit Kubernetes API activity?**
> Enable Kubernetes Audit Logging:
> - Every API request is logged with: user, timestamp, resource, action, result
> - Stored in files or sent to external systems
> Use for: Security forensics, compliance, detecting suspicious activity

---

**Q88. What is the CIS Kubernetes Benchmark?**
> CIS (Center for Internet Security) provides a set of security best practices for Kubernetes. Tools like **kube-bench** automatically check if your cluster follows these benchmarks.
```bash
kubectl apply -f kube-bench-job.yaml
kubectl logs job/kube-bench
```

---

## 7. kubectl Commands

**Q89. Essential kubectl commands every DevOps engineer must know.**
```bash
# Get resources
kubectl get pods
kubectl get pods -o wide                      # with node info
kubectl get pods -o yaml                      # full YAML output
kubectl get all -n production                 # all resources in namespace
kubectl get pods --watch                      # watch for changes

# Describe (detailed info + events)
kubectl describe pod mypod
kubectl describe node mynode
kubectl describe service myservice

# Logs
kubectl logs mypod
kubectl logs mypod -c container-name          # specific container
kubectl logs mypod --previous                 # previous container (after crash)
kubectl logs -l app=myapp                     # logs from all pods with label
kubectl logs mypod --since=1h                 # last 1 hour

# Execute
kubectl exec -it mypod -- bash
kubectl exec -it mypod -c sidecar -- sh       # specific container

# Apply / Delete
kubectl apply -f deployment.yaml
kubectl apply -f ./manifests/                 # apply directory
kubectl delete -f deployment.yaml
kubectl delete pod mypod --grace-period=0     # force delete

# Scale
kubectl scale deployment myapp --replicas=5

# Rollout
kubectl rollout status deployment/myapp
kubectl rollout history deployment/myapp
kubectl rollout undo deployment/myapp

# Port forward (for debugging)
kubectl port-forward pod/mypod 8080:80
kubectl port-forward service/myservice 8080:80

# Copy files
kubectl cp mypod:/app/logs/app.log ./app.log
kubectl cp ./config.yaml mypod:/app/config.yaml
```

---

**Q90. How do you troubleshoot a Pod that is in CrashLoopBackOff?**
```bash
# Step 1: Describe pod - check Events section
kubectl describe pod mypod

# Step 2: Check logs
kubectl logs mypod
kubectl logs mypod --previous   # logs from crashed container

# Step 3: Check resource limits (OOMKilled?)
kubectl describe pod mypod | grep -A 3 "Last State"

# Step 4: Try running with shell override
kubectl run debug --image=myapp --command -- sleep 3600
kubectl exec -it debug -- bash
# Now manually run your app command to see error

# Common causes:
# - Application error/exception on startup
# - Missing config/env variables
# - Can't connect to DB or dependency
# - OOMKilled (increase memory limit)
# - Wrong command/entrypoint
```

---

**Q91. What are Pod states and what do they mean?**
> | Status | Meaning |
> |---|---|
> | `Pending` | Pod accepted but not scheduled yet (no node, image pull pending) |
> | `Running` | Pod scheduled and containers running |
> | `Succeeded` | All containers completed successfully (Jobs) |
> | `Failed` | Container exited with non-zero code |
> | `CrashLoopBackOff` | Container repeatedly crashing, K8s backing off restarts |
> | `OOMKilled` | Killed because it exceeded memory limit |
> | `ImagePullBackOff` | Can't pull the container image |
> | `Terminating` | Pod being deleted |
> | `Unknown` | Can't communicate with the node |

---

**Q92. How do you get resource usage of Pods and Nodes?**
```bash
# Requires metrics-server installed
kubectl top pods
kubectl top pods -n production
kubectl top pods --sort-by=memory
kubectl top nodes
```

---

**Q93. What is `kubectl apply` vs `kubectl create`?**
> - `kubectl create` — Creates resource. Fails if it already exists.
> - `kubectl apply` — Creates if doesn't exist. Updates if exists. Preferred for GitOps/automation.
> - `kubectl replace` — Deletes and recreates resource.
> Always use `kubectl apply` in CI/CD pipelines.

---

**Q94. What is a kubeconfig file?**
> `~/.kube/config` stores cluster connection info — API server URL, certificates, contexts.
```bash
kubectl config get-contexts                   # list all contexts
kubectl config use-context production         # switch cluster
kubectl config current-context               # show current context
kubectl config set-context --current --namespace=production  # set default namespace
```

---

**Q95. How do you debug networking issues in Kubernetes?**
```bash
# Run a debug pod with network tools
kubectl run netdebug --image=nicolaka/netshoot -it --rm -- bash

# Inside the debug pod:
nslookup myservice              # test DNS resolution
curl http://myservice/health    # test HTTP connectivity
ping pod-ip                     # test pod connectivity
netstat -tlnp                   # check listening ports
traceroute myservice            # trace network path
```

---

## 8. Advanced & Real-World Scenarios

**Q96. What is GitOps and how does it work with Kubernetes?**
> GitOps uses Git as the single source of truth for Kubernetes infrastructure. The cluster state matches what's in Git.
> Tools: **ArgoCD**, **Flux**
> Workflow:
> 1. Developer pushes YAML changes to Git
> 2. ArgoCD detects the change
> 3. ArgoCD syncs the cluster to match Git state
> 4. Any manual kubectl changes are overwritten (Git wins)

---

**Q97. What is ArgoCD?**
> ArgoCD is a declarative GitOps CD tool for Kubernetes. It:
> - Watches a Git repo
> - Compares desired state (Git) vs actual state (cluster)
> - Syncs automatically or on approval
> - Provides a beautiful UI showing app health and sync status
> - Supports multi-cluster deployments

---

**Q98. How do you implement canary deployment in Kubernetes?**
```yaml
# Stable deployment (90% traffic)
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-stable
spec:
  replicas: 9
  template:
    metadata:
      labels:
        app: myapp
        version: stable

---
# Canary deployment (10% traffic)
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-canary
spec:
  replicas: 1    # 1 out of 10 total = 10% traffic
  template:
    metadata:
      labels:
        app: myapp
        version: canary

---
# Service selects BOTH by common label
spec:
  selector:
    app: myapp   # routes to both stable and canary pods
```
> For precise traffic splitting (e.g., exact 10%), use Istio VirtualService.

---

**Q99. How do you deploy to multiple Kubernetes clusters?**
> Options:
> 1. **Multiple kubeconfig contexts** — `kubectl --context=prod-cluster apply`
> 2. **ArgoCD ApplicationSet** — deploy same app to multiple clusters
> 3. **Helm + multiple values files** — `helm install -f values.prod.yaml`
> 4. **Flux multi-tenancy** — GitOps across clusters
> 5. **Rancher** — multi-cluster management UI

---

**Q100. How do you do zero-downtime deployment in Kubernetes?**
> 1. Set `strategy.type: RollingUpdate`
> 2. Set `maxUnavailable: 0` (never less than desired)
> 3. Set `maxSurge: 1` (one extra pod during update)
> 4. Implement proper **readinessProbe** (new pod only gets traffic when ready)
> 5. Set **preStop** hook for graceful shutdown:
```yaml
lifecycle:
  preStop:
    exec:
      command: ["sh", "-c", "sleep 5"]  # wait for in-flight requests
```
> 6. Set `terminationGracePeriodSeconds: 60`

---

**Q101. What is a Kubernetes Admission Controller?**
> Admission Controllers intercept API requests after authentication/authorization but before persistence. Built-in examples:
> - **NamespaceLifecycle** — rejects requests in terminating namespaces
> - **LimitRanger** — applies default limits
> - **ResourceQuota** — enforces quotas
> - **PodSecurity** — enforces pod security standards
> - **MutatingAdmissionWebhook** — custom mutation
> - **ValidatingAdmissionWebhook** — custom validation

---

**Q102. How do you handle database migrations in Kubernetes?**
> Option 1: Init container
```yaml
initContainers:
- name: run-migrations
  image: myapp:latest
  command: ["python", "manage.py", "migrate"]
  env:
  - name: DATABASE_URL
    valueFrom:
      secretKeyRef:
        name: db-secret
        key: url
```
> Option 2: Kubernetes Job (run once before deployment)
> Option 3: Migration as part of application startup (careful with multiple replicas)

---

**Q103. What is Velero?**
> Velero is a backup and restore tool for Kubernetes. It backs up:
> - Kubernetes resource definitions (YAML)
> - Persistent Volumes (snapshots)
> Restore entire cluster or specific namespaces after disaster.
```bash
velero backup create my-backup --include-namespaces production
velero restore create --from-backup my-backup
```

---

**Q104. What is cert-manager?**
> cert-manager automatically provisions and renews TLS certificates in Kubernetes. Integrates with Let's Encrypt, Vault, and other CAs.
```yaml
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: myapp-tls
spec:
  secretName: myapp-tls-secret
  issuerRef:
    name: letsencrypt-prod
    kind: ClusterIssuer
  dnsNames:
  - myapp.example.com
```

---

**Q105. How do you manage Kubernetes configuration across environments?**
> Options:
> 1. **Helm values files** — `values.dev.yaml`, `values.prod.yaml`
> 2. **Kustomize** — base + overlays per environment
> 3. **ArgoCD ApplicationSet** — matrix generator for environments
>
> Kustomize example:
> ```
> base/
>   deployment.yaml
>   service.yaml
> overlays/
>   dev/
>     kustomization.yaml  (replicas: 1, image: dev tag)
>   prod/
>     kustomization.yaml  (replicas: 10, image: prod tag)
> ```

---

**Q106. What is Kustomize?**
> Kustomize is a template-free way to customize Kubernetes YAML. It patches base configs without duplicating them.
```bash
kubectl apply -k overlays/production/
kubectl kustomize overlays/production/ | kubectl apply -f -
```

---

**Q107. What is the difference between Helm and Kustomize?**
> | Feature | Helm | Kustomize |
> |---|---|---|
> | Templating | Yes (Go templates) | No (patches/overlays) |
> | Package management | Yes (charts, versioning) | No |
> | Built into kubectl | No | Yes (`kubectl apply -k`) |
> | Learning curve | Steeper | Easier |
> | Best for | Distributing apps (third-party charts) | Environment-specific customization |
> Many teams use both: Helm for third-party apps, Kustomize for their own apps.

---

**Q108. How do you monitor a Kubernetes cluster?**
> Full observability stack:
> 1. **Prometheus** — scrapes metrics from pods, nodes, K8s components
> 2. **kube-state-metrics** — K8s object metrics (pod status, deployment status)
> 3. **node-exporter** — node hardware/OS metrics
> 4. **Grafana** — dashboards for all metrics
> 5. **Alertmanager** — alerts to Slack, PagerDuty, email
> 6. **Loki** — log aggregation
> 7. **Jaeger/Tempo** — distributed tracing
> Deploy with: `kube-prometheus-stack` Helm chart (everything in one chart).

---

**Q109. How do you upgrade a Kubernetes cluster safely?**
> 1. Read release notes and check breaking changes
> 2. Upgrade control plane first (one version at a time: 1.27→1.28→1.29)
> 3. Upgrade worker nodes (cordon → drain → upgrade → uncordon)
> 4. Upgrade add-ons (CoreDNS, kube-proxy, CNI plugin)
> 5. Upgrade kubectl to match cluster version
```bash
# Drain node safely before upgrade
kubectl cordon node-1           # mark unschedulable
kubectl drain node-1 --ignore-daemonsets --delete-emptydir-data
# ... upgrade node OS and kubelet ...
kubectl uncordon node-1         # mark schedulable again
```

---

**Q110. Real-world scenario: Design a production-grade Kubernetes setup on AWS EKS.**
```
INFRASTRUCTURE
├── EKS Control Plane (AWS managed, Multi-AZ)
├── Worker Node Groups
│   ├── On-Demand nodes (core services)
│   └── Spot nodes (batch/CI workers) - save 70%
└── AWS VPC with private subnets

NETWORKING
├── AWS VPC CNI (pods get VPC IPs)
├── AWS Load Balancer Controller (ALB Ingress)
├── External DNS (auto-update Route53)
└── cert-manager (auto TLS from ACM/Let's Encrypt)

SECURITY
├── IRSA (IAM Roles for Service Accounts) - no static keys
├── OPA Gatekeeper (policy enforcement)
├── Falco (runtime security)
├── Pod Security Standards (restricted namespace)
└── Sealed Secrets (encrypted secrets in Git)

DEPLOYMENTS
├── ArgoCD (GitOps - Git is source of truth)
├── Helm charts (packaging)
└── Kustomize (environment overlays)

AUTOSCALING
├── HPA (scale pods by CPU/custom metrics)
├── KEDA (event-driven scaling)
└── Cluster Autoscaler (scale nodes)

OBSERVABILITY
├── Prometheus + Grafana (metrics)
├── Loki (logs)
├── Jaeger (tracing)
└── PagerDuty (on-call alerting)

BACKUP & DR
├── Velero (cluster backup to S3)
├── Multi-AZ pods (PodAntiAffinity)
└── PodDisruptionBudgets (safe node operations)
```
---
---


