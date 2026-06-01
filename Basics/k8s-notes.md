# Kubernetes Notes

## What is a Namespace in Kubernetes?

A Namespace in Kubernetes is a logical partition inside a cluster used to organize and isolate resources.

It helps divide one Kubernetes cluster into multiple virtual environments.

---

## Docker vs Kubernetes — Explained in simple words

Docker is used for containerization, meaning packaging applications and their dependencies into portable containers.

Kubernetes is an orchestration platform that automates deployment, scaling, networking, and management of containers across a cluster of machines.

Docker solves the problem of running containers, while Kubernetes solves the problem of managing containers in production environments.

---

## What is kube-proxy in Kubernetes?

kube-proxy is a Kubernetes networking component that runs on each node and routes Service traffic to the appropriate Pods using networking rules like iptables or IPVS.

---

## Types of Services in Kubernetes and Their Roles

Kubernetes Services provide stable networking for Pods, and different Service types are used for internal communication, external access, cloud load balancing, and external DNS mapping.

### Service Types

#### ClusterIP

Exposes the application only inside the Kubernetes cluster for internal communication.

#### NodePort

Exposes the application externally using `<NodeIP>:<Port>`.

#### LoadBalancer

Exposes the application to the internet using a cloud provider’s external load balancer.

#### ExternalName

Maps a Kubernetes Service to an external DNS name instead of Pods.

---

## Difference Between NodePort and LoadBalancer Service

NodePort exposes applications externally through a node’s IP and port, while LoadBalancer provides a public external IP using a cloud provider’s load balancer for production-grade access.

---

## What is the Role of kubelet in Kubernetes?

kubelet is an agent that runs on every Kubernetes node and is responsible for managing Pods and containers on that node.

### Main Responsibilities

* Creates Pods
* Ensures containers are running
* Monitors container health
* Restarts failed containers

---

## Day-to-Day Activities as an AWS DevOps Engineer

As an AWS DevOps Engineer, my day-to-day activities involve:

* Managing CI/CD pipelines
* Monitoring infrastructure
* Handling deployments
* Troubleshooting production issues
* Automating repetitive tasks
* Ensuring application availability and scalability

### Common Tools

* Jenkins
* Docker
* Kubernetes
* Terraform
* AWS CloudWatch
* Prometheus
* Grafana

### Responsibilities

* Deploy applications
* Monitor infrastructure
* Manage EC2 instances
* Troubleshoot deployment failures
* Handle IAM permissions
* Optimize cloud costs
* Resolve environment and networking issues

---

## What is Ingress?

Ingress is a Kubernetes resource used to expose HTTP and HTTPS applications externally and route traffic to different Services based on paths or domain names.

It provides a single entry point and is managed by an Ingress Controller such as NGINX.

---

## What is SSL/TLS?

SSL/TLS secures communication between clients and servers by encrypting data and verifying server identity.

In Kubernetes, SSL is commonly implemented through Ingress Controllers or Load Balancers.

### SSL Modes

#### SSL Termination

Traffic is decrypted at the load balancer or ingress and forwarded internally as HTTP.

#### SSL Bridging

Traffic is decrypted and then re-encrypted before forwarding to backend servers.

#### SSL Passthrough

Encrypted traffic is forwarded directly to backend services without decryption.

### Best Practices

* Store certificates securely
* Protect private keys
* Use Kubernetes TLS Secrets
* Use AWS ACM when applicable

---

## Traditional Load Balancer vs Ingress Load Balancer

Traditional load balancers are hardware-based enterprise appliances designed for large-scale traffic management.

Ingress controllers such as NGINX are software-based reverse proxies and load balancers commonly used in cloud-native Kubernetes environments.

---

## Why was Ingress Introduced in Kubernetes?

Ingress was introduced to solve the problem of managing external HTTP/HTTPS access to multiple Kubernetes Services.

### Benefits

* Single entry point
* Centralized routing
* SSL/TLS handling
* Domain-based routing
* Path-based routing
* Reduced need for multiple LoadBalancers

---

## What is RBAC?

RBAC (Role-Based Access Control) is a Kubernetes authorization mechanism used to control access to cluster resources.

### Components

* Role
* ClusterRole
* RoleBinding
* ClusterRoleBinding

### Authentication Methods

* Client Certificates
* Bearer Tokens
* Service Accounts
* OIDC
* LDAP
* Webhooks

Authentication verifies identity, while RBAC determines permissions.

---

## What is OpenShift?

OpenShift is an enterprise Kubernetes platform with built-in DevOps, security, and management features.

---

## What is Argo CD?

Argo CD is a GitOps-based continuous delivery tool for Kubernetes that automatically synchronizes application manifests stored in Git repositories with Kubernetes clusters.

### Features

* Automated deployments
* Rollbacks
* Drift detection
* GitOps workflows

---

## What is Drift Detection?

Drift detection is the process of identifying differences between:

* Desired configuration stored in Git
* Actual running state in Kubernetes

Argo CD continuously monitors and reconciles these differences.

---

## What is a Custom Resource (CRD)?

Custom Resources (CRDs) extend Kubernetes by allowing users to define their own resource types beyond built-in objects.

CRDs are commonly used with Operators for automation.

---

## ConfigMaps vs Secrets

### ConfigMap

Stores non-sensitive configuration data.

### Secret

Stores sensitive data such as:

* Passwords
* Tokens
* Certificates

### Usage

#### Environment Variables

Used for simple configuration values.

#### Volume Mounts

Used for:

* Large configuration files
* Certificates
* Dynamic configuration reloads

---

## What is Prometheus?

Prometheus is a monitoring system used for collecting, storing, and querying metrics.

Helm is commonly used to deploy Prometheus in Kubernetes because it simplifies installation and lifecycle management.

---

# Deployment Strategies in Kubernetes

Deployment strategies define how new application versions are released while minimizing downtime and risk.

## 1. Rolling Update (Default)

Gradually replaces old Pods with new Pods without downtime.

## 2. Recreate

Terminates all old Pods before creating new Pods.

## 3. Blue-Green Deployment

Two environments exist simultaneously:

* Blue → Current production
* Green → New version

Traffic switches after validation.

## 4. Canary Deployment

Releases the new version to a small percentage of users before wider rollout.

## 5. A/B Testing

Routes users to different versions based on:

* Geography
* Headers
* User type
* Cookies

## 6. Shadow Deployment

Copies production traffic to a new version without impacting users.

---

## What is a Headless Service?

Headless Services are mainly used for StatefulSets and distributed applications where Pods require direct communication rather than load balancing through a ClusterIP.

---

## What is Kustomize?

Kustomize is used for environment-specific Kubernetes configuration management.

### Benefits

* Reusable base manifests
* Environment overlays
* No YAML duplication
* Easy management of dev, staging, and production environments

---

## What is Helm?

Helm is a package manager for Kubernetes.

### Benefits

* Reusable templates
* Easy configuration management
* Versioned releases
* Simple upgrades
* Rollback support

Helm is widely used to deploy and manage complex Kubernetes applications.
