# Trendyol-Devops-Case

## 1- Kubernetes cluster setup with Kubespray 

Preparation for Kubespray

```bash
sudo apt update -y 
sudo apt install ansible python3 python3-pip -y
```

Clone to Kubespray

```bash
git clone https://github.com/kubernetes-sigs/kubespray.git
```
Preinstall for Kubespray

```bash
cd kubespray
pip3 install -r requirements.txt 
pip3 install ruamel.yaml
cp -rfp inventory/sample inventory/mycluster
declare -a IPS=(192.168.0.247 192.168.0.248 192.168.0.249 192.168.0.250)
CONFIG_FILE=inventory/mycluster/hosts.yaml python3 contrib/inventory_builder/inventory.py ${IPS[@]}
```


### Configure to Kubespray

Configure Cluster (Cluster name ,CoreDNS , Calico)
```bash
vim inventory/mycluster/group_vars/k8s-cluster/k8s-cluster.yml
```

```bash
cat inventory/mycluster/hosts.yaml
```

```yaml
all:
  hosts:
    node1:
      access_ip: 192.168.0.249
      ansible_host: 192.168.0.249
      ip: 192.168.0.249
    node2:
      access_ip: 192.168.0.251
      ansible_host: 192.168.0.251
      ip: 192.168.0.251
    node3:
      access_ip: 192.168.0.250
      ansible_host: 192.168.0.250
      ip: 192.168.0.250
    node4:
      access_ip: 192.168.0.248
      ansible_host: 192.168.0.248
      ip: 192.168.0.248
  children:
    kube-master:
      hosts:
        node1:
    kube-node:
      hosts:
        node4:
        node3:
        node2:
        node1:
    etcd:
      hosts:
        node1:
    k8s-cluster:
      children:
        kube-node:
        kube-master:
    calico-rr:
      hosts: {}
```

Install Kubernetes with Kubespray

```bash
ansible-playbook -i inventory/mycluster/hosts.yaml cluster.yml
```

* server-0 --> node(master)
* server-1 --> node2
* server-2 --> node3
* server-3 --> node4

## 2- Prometheus , Consul ,Grafana  İnstallation

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

```yaml
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: kubernetes.io/hostname
                operator: In
                values:
                - node2              
```

Install Nginx Ingress

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v0.43.0/deploy/static/provider/cloud/deploy.yaml
```

Use Ingress
```bash
kubectl apply -f ingress.yaml
```
* Prometheus Dashboard
http://prometheus.serdarcan.com

-------------------------------------------------------------

Consul inject to pods. For automatic tracing add yaml (Prometheus)

```yaml
      annotations:
        prometheus.io/scrape: "true"
        prometheus.io/port: "9102"
        consul.hashicorp.com/connect-inject: "true"
```

See Grafana Dashboard logs on http://grafana.serdarcan.com  

<img align="center" width="750" height="450" src="https://github.com/serdarcanbigdereli/Trendyol-Devops-Case/blob/main/grafana.PNG">


## 3-EFK installation 

Elasticsearch , Fluentbit , Kibana , Kibana ingress , Elasticsearch HQ  Installation

```bash
kubectl apply -f efk-all.yaml
```

Added affinity to run on Node 4 

```yaml
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

See Kubernetes Cluster logs on http://kibana.serdarcan.com



## 4- Gitlab Installation CI/CD

Installation Gitlab

```bash
sudo apt update
sudo apt upgrade -y
sudo apt install -y ca-certificates curl openssh-server
curl -sS https://packages.gitlab.com/install/repositories/gitlab/gitlab-ce/script.deb.sh | sudo bash
sudo apt update
sudo apt -y install gitlab-ce
```

### CI/CD

You can add kubernetes annotations for prometheus tracing

```yaml
      annotations:
        prometheus.io/scrape: "true"
        prometheus.io/port: "9102"
        consul.hashicorp.com/connect-inject: "true"
```