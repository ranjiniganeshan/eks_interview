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
4. 
