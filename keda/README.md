# Deployment 
Deploy these Kubernetes Manifest chart for running KEDA (Kubernetes-based Event Driven Autoscaler) pods.

## Step 1: Install KEDA in Minikube Cluster
To deploy KEDA instaler, it is necessary to use helm chart as follows:

```
# Add the KEDA Helm repository
helm repo add kedacore https://kedacore.github.io/charts

# Update your Helm repository
helm repo update

# Install KEDA in the keda namespace
helm install keda kedacore/keda
```

## Step 2: Verify KEDA Installation
Ensure KEDA is properly installed:

```
# Check if KEDA pods are running
kubectl get pods 

# Verify CustomResourceDefinitions are created
kubectl get crd | grep keda
```

## Step 3: Implement the ScaledObjects for Each Deployment
The ScaledObjects files in this folder need to be executed to deploy KEDA resources as well as the services for each KEDA resources.
Execute these commands to deploy each YAML files.

```
kubectl apply -f redis-scaled-object.yaml
kubectl apply -f vote-scaled-object.yaml
kubectl apply -f result-scaled-object.yaml
kubectl apply -f worker-scaled-object.yaml
```

## Step 4: Verify the ScaledObjects Deployment
Check if all KEDA ScaledObjects were created successfully:

```
kubectl get scaledobjects -n default
```

Step 5: Check KEDA-created HPAs
KEDA creates Horizontal Pod Autoscalers (HPAs) based on your ScaledObjects. 
If deployment successfully implemented there should be several outputs. Let's verify they were created:

```
kubectl get hpa -n default
```

## Step 6: Monitor Autoscaling in Action (if necessary)
To see the autoscaling in action, monitor application pods and HPAs execute these commands:

```
# Watch pods scaling
kubectl get pods -n default -w

# Monitor HPA metrics
kubectl get hpa -n default -w
```

## Step 7: Troubleshooting
If issues are emerged during KEDA pods deployment it can be inspected using this command:

```
kubectl logs -n keda -l app.kubernetes.io/name=keda-operator
```

Also make sure thatthe Kubernetes metrics server addons is running in your Minikube cluster, as it's required for CPU-based scaling:

```
kubectl get deployment metrics-server -n kube-system
```

if not available execute this command

```
minikube addons enable metrics-server
```

There are error that usually occurs during deployment phase such with this output:

```
Error from server (Forbidden): error when creating "redis-scaled-object.yaml": admission webhook "vscaledobject.kb.io" denied the request: the scaledobject has a cpu trigger but the container redis doesn't have the cpu request defined
```

If such error output emerged, it is indicate that pods for app deployment do not have CPU resource requests defined.
It can be fixed by using this procedure:

### Step 1: Add Resource Requests to Pods App Deployments
This is necessary because Kubernetes calculates CPU utilization as a percentage of the requested CPU. 

* on Redis pod deployment:

```
$ kubectl patch deployment redis -n default --type=json -p '[
  {
    "op": "add", 
    "path": "/spec/template/spec/containers/0/resources", 
    "value": {
      "requests": {
        "cpu": "100m",
        "memory": "128Mi"
      },
      "limits": {
        "cpu": "200m",
        "memory": "256Mi"
      }
    }
  }
]'
```

* On Vote Pod Deployment

```
$ kubectl patch deployment vote -n default --type=json -p '[
  {
    "op": "add", 
    "path": "/spec/template/spec/containers/0/resources", 
    "value": {
      "requests": {
        "cpu": "100m",
        "memory": "128Mi"
      },
      "limits": {
        "cpu": "200m",
        "memory": "256Mi"
      }
    }
  }
]'
```

* On Result Pod Deployment

```
$ kubectl patch deployment result -n default --type=json -p '[
  {
    "op": "add", 
    "path": "/spec/template/spec/containers/0/resources", 
    "value": {
      "requests": {
        "cpu": "100m",
        "memory": "128Mi"
      },
      "limits": {
        "cpu": "200m",
        "memory": "256Mi"
      }
    }
  }
]'
```

* On Worker Pod Deployment

```
$ kubectl patch deployment worker -n default --type=json -p '[
  {
    "op": "add", 
    "path": "/spec/template/spec/containers/0/resources", 
    "value": {
      "requests": {
        "cpu": "100m",
        "memory": "128Mi"
      },
      "limits": {
        "cpu": "200m",
        "memory": "256Mi"
      }
    }
  }
]'
```

### Step 2: Verify Resource Requests
Check that the resource requests were successfully applied to all deployments:

```
# On Redis pod deployment
kubectl get deployment redis -n default -o jsonpath='{.spec.template.spec.containers[0].resources}'

# On Vote pod deployment
kubectl get deployment vote -n default -o jsonpath='{.spec.template.spec.containers[0].resources}'

# On Result pod deployment
kubectl get deployment result -n default -o jsonpath='{.spec.template.spec.containers[0].resources}'

# On Worker pod deployment
kubectl get deployment worker -n default -o jsonpath='{.spec.template.spec.containers[0].resources}'
```

### Step 3: Re-apply the ScaledObjects Manifest
The resource requests should be defined, after that, this step need to be executed to re-apply the ScaledObjects again:

```
kubectl apply -f redis-scaled-object.yaml
kubectl apply -f vote-scaled-object.yaml
kubectl apply -f result-scaled-object.yaml
kubectl apply -f worker-scaled-object.yaml
```

# Verifying KEDA Autoscaling
After applying resource requests and re-creating the ScaledObjects, check that they were created successfully:

```
# Check if ScaledObjects are created
kubectl get scaledobject -n default

# Check if HPAs are created by KEDA
kubectl get hpa -n default

# Describe example of ScaledObject to see details
kubectl describe scaledobject redis-scaler -n default
```