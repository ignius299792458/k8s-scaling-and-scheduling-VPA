# Kubernetes Vertical Pod Autoscaler (VPA)

The Vertical Pod Autoscaler (VPA) automatically adjusts the CPU and memory resource requests and limits for pods in your Kubernetes cluster. While Horizontal Pod Autoscaler (HPA) scales by changing the number of pod replicas, VPA scales by changing the resource allocation of individual pods.

## How VPA Works

1. VPA analyzes historical and current resource usage of pods
2. It recommends or automatically applies optimal CPU and memory settings
3. For automatic mode, it can evict pods to apply new resource settings
4. For recommendation mode, it only suggests changes without enforcing them

## Key Components

- **Recommender**: Monitors resource usage and calculates optimal resource requests
- **Updater**: Evicts pods that need to be restarted with new resource settings
- **Admission Controller**: Applies recommendations during pod creation

## Basic Example

Here's a simple VPA configuration:

```yaml
apiVersion: autoscaling.k8s.io/v1
kind: VerticalPodAutoscaler
metadata:
  name: my-app-vpa
spec:
  targetRef:
    apiVersion: "apps/v1"
    kind: Deployment
    name: my-app
  updatePolicy:
    updateMode: "Auto"
  resourcePolicy:
    containerPolicies:
    - containerName: '*'
      minAllowed:
        cpu: 100m
        memory: 50Mi
      maxAllowed:
        cpu: 1
        memory: 500Mi
      controlledResources: ["cpu", "memory"]
```

In this example:
- VPA targets a deployment named `my-app`
- `updateMode: "Auto"` means VPA will automatically apply changes
- Resource constraints ensure pods get at least 100m CPU and 50Mi memory
- Pods won't be allocated more than 1 CPU core and 500Mi memory

## VPA Update Modes

1. **Auto**: Automatically evicts pods and creates new ones with updated resource requests
2. **Recreate**: Similar to Auto but with all pods restarted at once
3. **Initial**: Only applies recommendations when pods are first created
4. **Off**: Only provides recommendations without applying them (monitoring only)

## Creating and Managing VPA

```bash
# Install VPA components (typically done at cluster level)
# kubectl apply -f https://github.com/kubernetes/autoscaler/tree/master/vertical-pod-autoscaler/deploy

# Create VPA from YAML file
kubectl apply -f vpa.yaml

# Check VPA status
kubectl get vpa
kubectl describe vpa my-app-vpa
```

## Getting VPA Recommendations

To see what the VPA is recommending without applying changes:

```yaml
apiVersion: autoscaling.k8s.io/v1
kind: VerticalPodAutoscaler
metadata:
  name: my-app-vpa
spec:
  targetRef:
    apiVersion: "apps/v1"
    kind: Deployment
    name: my-app
  updatePolicy:
    updateMode: "Off"
```

Then check recommendations with:

```bash
kubectl describe vpa my-app-vpa
```

## Advanced Features

### Container-Specific Policies

You can set different policies for different containers:

```yaml
resourcePolicy:
  containerPolicies:
  - containerName: 'app'
    minAllowed:
      cpu: 200m
      memory: 100Mi
  - containerName: 'sidecar'
    minAllowed:
      cpu: 50m
      memory: 20Mi
```

### Excluding Containers

You can exclude certain containers from VPA management:

```yaml
resourcePolicy:
  containerPolicies:
  - containerName: 'istio-proxy'
    mode: "Off"
```

## Best Practices

1. Start with `updateMode: "Off"` to observe recommendations before applying
2. Set reasonable minimum and maximum resource boundaries
3. Consider pod disruption budgets when using automatic updates
4. Use VPA with applications that can handle restarts gracefully
5. Combine with HPA appropriately - typically use VPA for CPU/memory efficiency and HPA for handling load variations
6. Consider using `Initial` mode for stability-critical applications

## Limitations

1. VPA cannot be used with HPA based on CPU or memory
2. Pod restarts are necessary to apply new resource settings
3. Applications must be designed to handle restarts gracefully
4. Recommendations might fluctuate if workload patterns change frequently

VPA is particularly valuable for applications with unpredictable resource needs, newly deployed applications with uncertain requirements, and batch jobs with varying computational demands.
