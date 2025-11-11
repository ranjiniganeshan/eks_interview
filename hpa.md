### Problem statement 1:  How does hpa work?

##### prequesite
1. eks cluster creation using create_cluster.md
2. install metrics server
```
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```
3. verify if the metrics server status is true
```
kubectl get apiservice v1beta1.metrics.k8s.io -o yaml | grep "status:"

status: "True"
```
4. create a file hpa-demo.yaml 
```
kubectl apply -f hpa-demo.yaml
```
5. Create a Horizontal Pod Autoscaler
```
kubectl autoscale deployment php-apache \
  --cpu-percent=50 \
  --min=1 \

  
  --max=10
```
or 
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: php-apache
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
        image: k8s.gcr.io/hpa-example
        ports:
        - containerPort: 80
        resources:
          requests:
            cpu: 200m
          limits:
            cpu: 500m
---
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: php-apache
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: php-apache
  minReplicas: 1
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 50

```
6.
  ```
  kubectl get hpa
  ```
7. Generate Load to Trigger Scaling
```
kubectl run -i --tty load-generator --image=busybox /bin/sh
Inside the shell:
```
while true; do wget -q -O- http://php-apache.default.svc.cluster.local; done
```
8. Observer the autoscaling in action
```
kubectl get hpa php-apache --watch
```

```
ranjiniganeshan@Ranjinis-MacBook-Pro patch_manager % kubectl get hpa php-apache --watch

NAME         REFERENCE               TARGETS       MINPODS   MAXPODS   REPLICAS   AGE
php-apache   Deployment/php-apache   cpu: 0%/50%   1         10        1          43s
php-apache   Deployment/php-apache   cpu: 222%/50%   1         10        1          60s
php-apache   Deployment/php-apache   cpu: 249%/50%   1         10        4          75s
php-apache   Deployment/php-apache   cpu: 114%/50%   1         10        5          90s
php-apache   Deployment/php-apache   cpu: 52%/50%    1         10        5          105s
php-apache   Deployment/php-apache   cpu: 70%/50%    1         10        5          2m
php-apache   Deployment/php-apache   cpu: 55%/50%    1         10        7          2m15s
php-apache   Deployment/php-apache   cpu: 45%/50%    1         10        7          2m30s
php-apache   Deployment/php-apache   cpu: 49%/50%    1         10        7          2m45s
php-apache   Deployment/php-apache   cpu: 52%/50%    1         10        7          3m
php-apache   Deployment/php-apache   cpu: 44%/50%    1         10        7          3m15s
php-apache   Deployment/php-apache   cpu: 47%/50%    1         10        7          3m30s
php-apache   Deployment/php-apache   cpu: 46%/50%    1         10        7          3m45s
php-apache   Deployment/php-apache   cpu: 50%/50%    1         10        7          4m
php-apache   Deployment/php-apache   cpu: 42%/50%    1         10        7          4m15s
php-apache   Deployment/php-apache   cpu: 42%/50%    1         10        7          4m30s
php-apache   Deployment/php-apache   cpu: 51%/50%    1         10        7          4m45s
php-apache   Deployment/php-apache   cpu: 47%/50%    1         10        7          5m
php-apache   Deployment/php-apache   cpu: 45%/50%    1         10        7          5m30s
php-apache   Deployment/php-apache   cpu: 54%/50%    1         10        7          5m45s
php-apache   Deployment/php-apache   cpu: 41%/50%    1         10        7          6m
php-apache   Deployment/php-apache   cpu: 47%/50%    1         10        7          6m15s
php-apache   Deployment/php-apache   cpu: 48%/50%    1         10        7          6m30s
php-apache   Deployment/php-apache   cpu: 48%/50%    1         10        7          6m45s
php-apache   Deployment/php-apache   cpu: 50%/50%    1         10        7          7m
php-apache   Deployment/php-apache   cpu: 45%/50%    1         10        7          7m15s
php-apache   Deployment/php-apache   cpu: 50%/50%    1         10        7          7m30s
php-apache   Deployment/php-apache   cpu: 47%/50%    1         10        7          7m45s
php-apache   Deployment/php-apache   cpu: 49%/50%    1         10        7          8m
php-apache   Deployment/php-apache   cpu: 44%/50%    1         10        7          8m15s
php-apache   Deployment/php-apache   cpu: 48%/50%    1         10        7          8m30s
php-apache   Deployment/php-apache   cpu: 47%/50%    1         10        7          8m45s
php-apache   Deployment/php-apache   cpu: 47%/50%    1         10        7          9m
php-apache   Deployment/php-apache   cpu: 50%/50%    1         10        7          9m30s
php-apache   Deployment/php-apache   cpu: 45%/50%    1         10        7          9m46s
php-apache   Deployment/php-apache   cpu: 49%/50%    1         10        7          10m
php-apache   Deployment/php-apache   cpu: 42%/50%    1         10        7          10m
php-apache   Deployment/php-apache   cpu: 47%/50%    1         10        7          10m
php-apache   Deployment/php-apache   cpu: 49%/50%    1         10        7          10m
php-apache   Deployment/php-apache   cpu: 50%/50%    1         10        7          11m
php-apache   Deployment/php-apache   cpu: 48%/50%    1         10        7          11m
php-apache   Deployment/php-apache   cpu: 45%/50%    1         10        7          11m
php-apache   Deployment/php-apache   cpu: 48%/50%    1         10        7          11m
php-apache   Deployment/php-apache   cpu: 47%/50%    1         10        7          12m
php-apache   Deployment/php-apache   cpu: 49%/50%    1         10        7          12m
php-apache   Deployment/php-apache   cpu: 41%/50%    1         10        7          12m
php-apache   Deployment/php-apache   cpu: 51%/50%    1         10        7          13m

```

obeservation:

* Pods automatically scaling from 1 → N (based on CPU usage)

* Then scaling back down when load decreases

* Verified via both HPA metrics and kubectl get pods

example: 
```
php-apache   Deployment/php-apache   cpu: 222%/50%   1   10   1   60s
```

#### CPU usage has spiked to 222%, well above 50%.

* HPA starts calculating a new desired replica count.
```
desiredReplicas = currentReplicas × ( currentMetricValue / targetValue )

#### Example:
CPU usage has spiked to **222%**, well above the target **50%**.

So:
desiredReplicas = 1 × (222 / 50)
desiredReplicas = 4.44 ≈ 4 replicas (rounded)


➡️ The HPA will scale the `php-apache` deployment from **1** replica to **4** replicas to handle the higher CPU load.



