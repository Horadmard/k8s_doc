# CBA Web App Deployment with Helm and Kubernetes

## Prerequisites

Machines/VMs: At least one control-plane node and one or more worker nodes, each running Linux (Ubuntu 22.04+ recommended).

Access: SSH access to all nodes with a user having sudo privileges.

Network: Ensure all nodes can communicate over required ports (e.g., 6443, 2379–2380, 10250, 10251, 10252).

### 1. Setting Up the Kubernetes Cluster

#### 1.1 Initialize the Control-Plane
```
sudo snap install k8s --classic --channel=1.32-classic/stable
sudo k8s bootstrap
# sudo k8s bootstrap --help   # For custom configurations
sudo k8s status --wait-ready  # Check cluster status
```
#### 1.2 Configure the Cluster

Enable networking (Cilium), CoreDNS, and local storage (EBS):
```
sudo k8s enable network dns local-storage
# sudo k8s set local-storage.local-path=/path/to/new/folder   # Specify local storage path
```
#### 1.3 Join Worker Nodes

On the control-plane node:
```
sudo k8s get-join-token worker --worker
```
On each worker node:
```
sudo k8s join-cluster <join-token>
```
#### 1.4 Alias and Auto-Completion (Optional)

Add the following to simplify commands:
```
alias k='sudo k8s kubectl'
echo "source <(kubectl completion bash)" >> ~/.bashrc
source ~/.bashrc
```
### 2. Cloning and Configuring Helm Charts

#### 2.1 Clone the Project Repository
```
git clone https://git.hyvatech.com/devops/cba.git
cd cba
```
#### 2.2 Edit values.yaml

Customize the values.yaml file based on the following structure:
```
replicaCount: 1
image:
  repository: cba
  pullPolicy: IfNotPresent
  tag: ""
imagePullSecrets: []
nameOverride: ""
fullnameOverride: ""
serviceAccount:
  create: true
  annotations: {}
  name: ""
podAnnotations: {}
podSecurityContext: {}
securityContext: {}
service:
  type: ClusterIP
  port: 80
resources: {}
nodeSelector: {}
tolerations: []
affinity: {}
```
Adjust values such as replicaCount, image.repository, service.type, and other fields according to your deployment needs.

### 3. Installing Helm and Deploying the Web App

#### 3.1 Install the Helm CLI
```
curl https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3 | bash
helm version
```
#### 3.2 Create a Namespace (Optional)
```
sudo k8s kubectl create namespace web-app
```
If your charts depend on subcharts, update dependencies:
```
helm dependency update .
```
#### 3.3 Deploy the Application with Helm
```
helm install web-app \
  ./helm/web-app \
  --namespace web-app \
  --values ./helm/web-app/values.yaml
```
To upgrade after making changes:
```
helm upgrade web-app \
  ./helm/web-app \
  --namespace web-app \
  --values ./helm/web-app/values.yaml
```
### 4. Monitoring and Accessing Your Application

#### 4.1 Verify Deployment

# Check Helm releases:
```
helm list --namespace web-app
```
# Check pods and services:
```
kubectl get pods --namespace web-app
kubectl get svc --namespace web-app
```
#### 4.2 Inspect Logs and Events

## Logs of a specific pod:
kubectl logs web-app-abc123 -n web-app

## Describe resources to view events:
```
kubectl describe pod web-app-abc123 -n web-app
```
#### 4.3 Port-Forwarding (Local Testing)
```
kubectl port-forward svc/web-app 8080:80 -n web-app
# Then open http://localhost:8080 in your browser
```
#### 4.4 Accessing via Ingress or LoadBalancer

If you enabled an Ingress controller or LoadBalancer, retrieve the endpoint:
```
kubectl get ingress -n web-app
# or
kubectl get svc -n web-app | grep LoadBalancer
```
#### 4.5 Monitoring with Lens

Lens offers a powerful graphical interface for Kubernetes cluster monitoring and management. To integrate Lens into your workflow:

Download & InstallVisit the Lens website and download the package for your operating system (Windows, macOS, or Linux).

Add Your Cluster

Open Lens and click + Add Cluster.

Choose Custom KubeConfig or Select from existing contexts.

Point Lens to your $HOME/.kube/config file or specify the context used for the web-app namespace.

Explore ResourcesNavigate through Namespaces, Nodes, Workloads (Deployments, DaemonSets, StatefulSets), Services, ConfigMaps, and more. Lens provides live status updates and resource counts.

View MetricsLens utilizes the Kubernetes Metrics Server to show CPU and memory charts. Click on a Node or Pod to view resource utilization over time.

Inspect LogsStream real-time Pod logs, filter outputs, and troubleshoot faster directly within Lens.

Terminal AccessUse the built-in terminal to run commands or kubectl exec into any Pod without switching to your CLI.

Lens significantly streamlines Kubernetes cluster management through visualizations and intuitive tooling.
