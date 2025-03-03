# K8s-WordPress
This repository contains two Helm charts for deploying WordPress, MySQL, Prometheus, and NGINX Ingress.  
There are two variations of the deployment:

1. **`wordpress-and-monitoring-elb`** - Uses an AWS Elastic Load Balancer (ELB) for external access.
2. **`wordpress-and-monitoring-ingress`** - Uses an NGINX Ingress Controller for routing traffic.

## 📂 Repository Structure


## Deployment Instructions

### Prerequisites
- Kubernetes cluster (EKS, Minikube, or other)
- Helm installed (`v3.x`)
- Namespace `meitaltal` created in the cluster:
```sh
  kubectl create namespace meitaltal
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
<<<<<<< HEAD
=======

>>>>>>> 2177381af5694e05fc157aef147f0b6ec816c383
