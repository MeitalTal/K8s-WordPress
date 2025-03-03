# K8s-WordPress
This repository contains two Helm charts for deploying WordPress, MySQL, Prometheus, and NGINX Ingress.  
There are two variations of the deployment:

1. **`wordpress-and-monitoring-elb`** - Uses an AWS Elastic Load Balancer (ELB) for external access.
2. **`wordpress-and-monitoring-ingress`** - Uses an NGINX Ingress Controller for routing traffic.

## Deployment Instructions

### Clone 
```sh
  git clone https://github.com/MeitalTal/K8s-WordPress.git
  cd K8s-WordPress
```

### Prerequisites
- Kubernetes cluster (EKS, Minikube, or other)
- AWS configure
- Helm installed (`v3.x`)
```sh
  curl https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3 > get_helm.sh
  chmod 700 get_helm.sh
  ./get_helm.sh
```
- Namespace `meitaltal` created in the cluster:
```sh
  kubectl create namespace meitaltal
```
- create your ECR secret:
```sh
  kubectl create secret docker-registry ecr-secret \
  --docker-server=992382545251.dkr.ecr.us-east-1.amazonaws.com \
  --docker-username=AWS \
  --docker-password=$(aws ecr get-login-password --region us-east-1) \
  --namespace meitaltal
```

### Deploying with ELB
This deployment creates an Elastic Load Balancer (ELB) to expose WordPress.
```sh
  helm install wordpress-elb ./wordpress-and-monitoring-elb -n meitaltal
```
Verify the deployment:
```sh
  kubectl get svc -n meitaltal
```
Look for the EXTERNAL-IP of the LoadBalancer and use it to access WordPress/Grafana.

### Deploying with Ingress
This deployment creates an Elastic Load Balancer (ELB) to expose WordPress.
```sh
  helm install wordpress-ingress ./wordpress-and-monitoring-ingress -n meitaltal
```
Verify the deployment:
```sh
  kubectl get ingress -n meitaltal
```
**Make sure the host matches your domain or public IP.** 

### Deploying without helm 
Install NGINX Ingress Controller:
```sh
  helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
  helm repo update
  helm install meitaltal-ingress ingress-nginx/ingress-nginx --namespace meitaltal  --set controller.ingressClassResource.name=meitaltal
```
Apply the yaml files:
```sh
  kubectl apply -f manifests/mysql-secret.yaml
  kubectl apply -f manifests/mysql-statefulset.yaml
  kubectl apply -f manifests/mysql-pv-pvc.yaml
  kubectl apply -f manifests/mysql-service.yaml 
  kubectl apply -f manifests/wordpress-deployment.yaml
  kubectl apply -f manifests/wordpress-pv-pvc.yaml
  kubectl apply -f manifests/wordpress-ingress.yaml
  kubectl apply -f manifests/wordpress-service.yaml
```
Install kube-prom-stack:
```sh
  helm repo add prometheus-community https://prometheus-community.github.io/helm-charts 
  helm repo update
  helm install meitaltal-kube-prom-stack prometheus-community/kube-prometheus-stack --namespace meitaltal 
  kubectl apply -f manifests/kube-prom-stack-grafana.yaml # if you want to access grafana via ELB 
```

**Make sure the host matches your domain or public IP.** 

### Access Grafana:
1. Log in to Grafana
Open a web browser and access the Grafana instance using the appropriate URL (LoadBalancer/Ingress URL).

The default login credentials are:
Username: admin
Password: prom-operator

2. Navigate to the Dashboards Section
Once logged in, on the left-hand menu, click on the "+" icon to expand more options and then click on "Import".

3. Import the Dashboard - the file is "Kubernetes _ Compute Resources _ Namespace (Pods).json"


# ⚙️ Configuration 
Modify values.yaml to customize your deployment.
If you change the namespace in `values.yaml`, make sure to update the namespace in all commands in this guide. 
# Uninstall 
```sh
  helm uninstall wordpress-elb -n meitaltal
  helm uninstall wordpress-ingress -n meitaltal
```
# 🔧 Troubleshooting
1. Prometheus is configured to scrape metrics from Node Exporter on port 9100.
If you encounter issues with Prometheus not collecting metrics, follow these steps:
Check if Prometheus is running:
```sh
kubectl get pods -n meitaltal | grep prometheus
```
If the pod is not running, check logs:
```sh
kubectl logs <prometheus-pod-name> -n meitaltal
```
Edit the DaemonSet (if needed):
If there is a port conflict, update the kube-prometheus-stack DaemonSet:
```sh
kubectl edit daemonset.apps/meitaltal-kube-prom-stack-prometheus-node-exporter
```
Find and modify 9100 port and save and exit the editor.
Restart Prometheus Components:
```sh
kubectl rollout restart daemonset meitaltal-kube-prom-stack-prometheus-node-exporter -n meitaltal
```
