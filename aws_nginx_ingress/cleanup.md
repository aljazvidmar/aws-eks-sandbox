# CLEANUP

```bash
# delete cheese app namespace
kubectl delete all --all -n eks-cheese-app-nginx
kubectl delete namespace eks-cheese-app-nginx

# delete ingress-nginx
kubectl delete all --all -n ingress-nginx
kubectl delete namespace ingress-nginx

# delete cluster and other related AWS resources
eksctl delete cluster -f eks-cluster.yaml
```