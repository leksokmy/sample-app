# Prerequisites
Tools that needed to run Kubernetes Manifest for monitoring namespaces.

* kubectl (If already installed skipp it)
* Helm (the package manager for Kubernetes)

## Step 1: Helm Install

Follow this steps to install helm chart

```
# Linux
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
chmod 700 get_helm.sh
./get_helm.sh
```

## Step 2: Add Prometheus Community Helm Repository
Add the repository in order the helm command can obtain installer packages and then update it.

```
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo add grafana https://grafana.github.io/helm-charts
helm repo update
```

## Step 3: Create a Monitoring Namespace
A specific namespaces is needed to isolate env in Kubernetes cluster.

```
kubectl create namespace monitoring
```

## Step 4: Deploy Prometheus using Helm
Execute prometheus-values.yaml file with following command:

```
helm install prometheus prometheus-community/prometheus --namespace monitoring --values prometheus-values.yaml
```

## Step 5: Deploy Grafana using Helm
Execute grafana-values.yaml file with following command:

```
helm install grafana grafana/grafana --namespace monitoring --values grafana-values.yaml
```

## Step 6: Configure Service Monitor for Deployments env
To monitor your specific deployments (redis, result, vote, worker), create a ServiceMonitor resource. Execute this below command to apply config :

```
kubectl apply -f service-monitors.yaml
```

## Check services
To ensure that pods can obtain services. It is necessary to check every deployed pods services

```
kubect get svc -o wide -n monitoring
```