
# Kubernetes Lab Task - Full-Stack Cloud Programming

## **Namespace Creation**
The task requires creating all resources within a namespace called `zad1`.

### **Manifest for Namespace**
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: zad1
```

Command:
```bash
kubectl apply -f namespace.yaml
```

---

## **Resource Quotas**
Resource quotas for the namespace were defined as:
- Maximum Pods: 10
- CPU: 2000m (2 CPU)
- Memory: 1.5Gi

### **Manifest for Resource Quota**
```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: resource-quota
  namespace: zad1
spec:
  hard:
    pods: "10"
    requests.cpu: "2000m"
    requests.memory: "1.5Gi"
    limits.cpu: "2000m"
    limits.memory: "1.5Gi"
```

Command:
```bash
kubectl apply -f quota.yaml
```

---

## **Worker Pod**
A single Pod named `worker` with specific resource requests and limits.

### **Manifest for Worker Pod**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: worker
  namespace: zad1
spec:
  containers:
  - name: nginx-container
    image: nginx
    resources:
      requests:
        memory: "100Mi"
        cpu: "100m"
      limits:
        memory: "200Mi"
        cpu: "200m"
```

Command:
```bash
kubectl apply -f worker-pod.yaml
```

---

## **Deployment and Service**
A deployment using the `php-apache` container image with autoscaling support.

### **Manifest for Deployment**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: php-apache
  namespace: zad1
spec:
  replicas: 1
  selector:
    matchLabels:
      app: php-apache
  template:
    metadata:
      labels:
        app: php-apache
    spec:
      containers:
      - name: php-apache
        image: php:7.4-apache
        resources:
          requests:
            memory: "150Mi"
            cpu: "150m"
          limits:
            memory: "250Mi"
            cpu: "250m"
```

### **Manifest for Service**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: php-apache-service
  namespace: zad1
spec:
  selector:
    app: php-apache
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
```

Command:
```bash
kubectl apply -f php-apache-deployment.yaml
kubectl apply -f php-apache-service.yaml
```

---

## **Horizontal Pod Autoscaler**
HPA was configured with the following parameters:
- `minReplicas`: 1
- `maxReplicas`: 5
- `targetCPUUtilizationPercentage`: 50

### **Manifest for HPA**
```yaml
apiVersion: autoscaling/v1
kind: HorizontalPodAutoscaler
metadata:
  name: php-apache-hpa
  namespace: zad1
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: php-apache
  minReplicas: 1
  maxReplicas: 5
  targetCPUUtilizationPercentage: 50
```

Command:
```bash
kubectl apply -f hpa.yaml
```

---

## **Autoscaling Test**
A load generator was created to simulate traffic for the `php-apache` deployment.

Command:
```bash
kubectl run -i --tty load-generator --image=busybox --restart=Never -- /bin/sh -c "while true; do wget -q -O- http://php-apache-service.zad1.svc.cluster.local; done"
```

HPA results:
```plaintext
Name:                   php-apache-hpa
Namespace:              zad1
Min replicas:           1
Max replicas:           5
Desired replicas:       3
Current CPU utilization: 60%
```

---

## **Non-Mandatory Task**
### **1. Can you update a deployment controlled by HPA?**
**Answer**: **Yes**, it is possible. HPA works independently from the Deployment controller. 

**Source**: [Kubernetes - Horizontal Pod Autoscaler](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/).

### **2. Rolling Update Parameters**
To ensure at least 2 Pods are active during the update without exceeding quotas:

**Rolling Update Configuration**:
```yaml
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxUnavailable: 0
    maxSurge: 1
```

Changes in HPA:
- Adjust `maxReplicas` to `6` to accommodate an additional Pod during the update.

**Justification**:
- `maxUnavailable: 0` ensures no downtime.
- `maxSurge: 1` allows a single additional Pod during updates, adhering to resource limits.

---

## **Summary**
This task demonstrated creating and managing Kubernetes objects under specific constraints, including resource quotas and autoscaling. Autoscaler behavior and resource limits were validated through tests.
