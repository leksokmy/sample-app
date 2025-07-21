## Step 1: Create Namespace
A specific namespace for loggin is intended to isolate pods, so that pods can be recognized easily

```
kubectl create namespace logging
```

## Step 2: Deploy Elasticsearch
Elasticsearch will store all the logs collected from your cluster. Execute a file named elasticsearch.yaml with the following content:

```
kubectl apply -f elasticsearch.yaml -n logging
```

## Step 3: Deploy Kibana
Kibana provides a UI for visualizing the namescape logs. Logging is needed to be visualized in web form. Execute the kibana.yaml file with following command:

```
kubectl apply -f kibana.yaml
```

## Step 4: Deploy Fluentd
Fluentd will collect logs from all containers. Execute a file named fluentd.yaml with following command:

```
kubectl apply -f fluentd.yaml
```

## Step 5: Verify the Deployment
Check if all components on pods are running:

```
kubectl get pods -n logging
```

## Step 6: Deploy Fluentd Configmap
Fluentd deployment is set up to collect container logs, it is able to collect metrics as well. Execute the file with following command :

```
kubectl apply -f fluentd-configmap.yaml
```

## Step 7: Deploy Metrics Exporter
To collect more detailed metrics about in the Minikube cluster, the Prometheus Node Exporter need to be deployed. Execute this command:

```
kubectl apply -f node-exporter.yaml -n logging
```