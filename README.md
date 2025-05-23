# Prerequisites

Machines/VMs: At least one control-plane node and one or more worker nodes, each running Linux (Ubuntu 22.04+ recommended).

Access: SSH access to all nodes, with a user having sudo privileges.

Network: Ensure all nodes can communicate over required ports (e.g., 6443, 2379–2380, 10250, 10251, 10252).

git (to clone the charts repository).

## 1. Setting Up the Canonical Kubernetes Cluster

### 1.1 Initialize cluster on the Control-Plane

```
sudo snap install k8s --classic --channel=1.32-classic/stable
sudo k8s bootstrap
# sudo k8s bootstrap --help ## for custom configurations
sudo k8s status --wait-ready ## check the cluster status
```

### 1.2 Configure Cluster for hosting app:
```
sudo k8s enable network, dns, local-storage
```

### 1.3 Join Worker Nodes

On each worker node, run the kubeadm join command printed by kubeadm init. It looks like this:
```
sudo kubeadm join <CONTROL_PLANE_IP>:6443 \
  --token abcdef.0123456789abcdef \
  --discovery-token-ca-cert-hash sha256:...
```

Verify all nodes are Ready:
```
kubectl get nodes
```

## 2. Cloning and Configuring Helm Charts

### 2.1 Clone the Project Repository

# On your local machine or control-plane
```
git clone https://git.hyvatech.com/devops/cba.git
cd cba
```

If your charts depend on subcharts, update dependencies:
```
helm dependency update .
```
### 2.2 Edit values.yaml

Open the values.yaml file and set values for your environment:

```
replicaCount: 3
image:
  repository: your-registry/web-app
  tag: v1.2.3
service:
  type: LoadBalancer
  port: 80
ingress:
  enabled: true
  host: webapp.example.com
```

Adjust resource requests, environment variables, and any custom configuration sections as needed.

## 3. Installing Helm and Deploying the Web App

### 3.1 Install Helm CLI

Download and install the latest Helm version:
```
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
helm version
```

### 3.2 Create a Namespace (Optional)
```
kubectl create namespace web-app
```

### 3.3 Deploy with Helm
```
helm install web-app \
  ./helm/web-app \
  --namespace web-app \
  --values ./helm/web-app/values.yaml
```

To upgrade after changes:
```
helm upgrade web-app \
  ./helm/web-app \
  --namespace web-app \
  --values ./helm/web-app/values.yaml
```

## 4. Monitoring and Accessing Your Application

### 4.1 Verify Deployment

# Check Helm releases:
```
helm list --namespace web-app
```
# Check pods and services:
```
kubectl get pods --namespace web-app
kubectl get svc --namespace web-app
```
### 4.2 Inspect Logs and Events

# Logs of a failing pod:
kubectl logs web-app-abc123 -n web-app

# Describe resources to view events:
kubectl describe pod web-app-abc123 -n web-app

4.3 Port-Forwarding (Local Testing)

kubectl port-forward svc/web-app 8080:80 -n web-app
## Then open http://localhost:8080 in your browser

4.4 Using an Ingress or LoadBalancer

If you enabled an Ingress controller or LoadBalancer, point your DNS or access the external IP:

kubectl get ingress -n web-app
## or
kubectl get svc -n web-app | grep LoadBalancer

Additional Considerations

TLS/SSL: Integrate Cert-Manager for automatic TLS certificates.

RBAC: Create service accounts and restrict permissions in production.

CI/CD: Automate Helm chart linting (helm lint) and deployment via pipelines.

Helm Tests: Define tests in templates/tests/ and run helm test web-app.

