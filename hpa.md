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
6. ``` kubectl get hpa ```

  


