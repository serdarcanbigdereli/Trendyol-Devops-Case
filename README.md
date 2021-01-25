# Trendyol-Devops-Case

1- Kubernetes cluster setup with Kubespray 

Preparation for Kubespray

```bash
sudo apt update -y 
sudo apt install ansible python3 python3-pip -y
```

Clone to Kubespray

```bash
git clone https://github.com/kubernetes-sigs/kubespray.git
```
install Kubernetes Cluster

```bash
cd kubespray
pip3 install -r requirements.txt 
pip3 install ruamel.yaml
cp -rfp inventory/sample inventory/mycluster
declare -a IPS=(192.168.0.247 192.168.0.248 192.168.0.249 192.168.0.250)
CONFIG_FILE=inventory/mycluster/hosts.yaml python3 contrib/inventory_builder/inventory.py ${IPS[@]}
ansible-playbook -i inventory/mycluster/hosts.yaml cluster.yml
```
* server-0 --> node(master)
* server-1 --> node2
* server-2 --> node3
* server-3 --> node4

2- Prometheus , Consul ,Grafana  İnstallation

I used Layer 7 Observability with Consul Service Mesh, Prometheus, Grafana, and Kubernetes

Helm3 İnstallation

```bash
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3
chmod 700 get_helm.sh
./get_helm.sh
```
Add Helm repo

```bash

helm repo add prometheus-community https://prometheus-community.github.io/helm-charts && \
helm repo add grafana https://grafana.github.io/helm-charts && \
helm repo add hashicorp https://helm.releases.hashicorp.com && \
helm repo update

```
Clone the GitHub repository

```bash
git clone https://github.com/hashicorp/learn-consul-kubernetes.git
```

Deploy Consul , Prometheus , Grafama

```bash
helm install -f layer7-observability/helm/consul-values.yaml consul hashicorp/consul --version "0.27.0" --wait
helm install -f layer7-observability/helm/prometheus-values.yaml prometheus prometheus-community/prometheus --version "11.7.0" --wait
helm install -f layer7-observability/helm/grafana-values.yaml grafana grafana/grafana --version "5.3.6" --wait
```
Edit Consul , Promethes ,Grafana Deployment for node affinity

```bash
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: kubernetes.io/hostname
                operator: In
                values:
                - node4                
```

Install Nginx Ingress

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v0.43.0/deploy/static/provider/cloud/deploy.yaml
```

Use Ingress
```bash
kubectl apply -f ingress.yaml
```
