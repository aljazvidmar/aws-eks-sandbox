# CLEANUP

```bash
# delete cheese app namespace
kubectl delete all --all -n eks-cheese-app-nginx
kubectl delete namespace eks-cheese-app-nginx

# delete aws load balancer controller
helm uninstall aws-load-balancer-controller -n kube-system

# delete aws IAM service account
eksctl delete iamserviceaccount \
    --cluster ${CLUSTER_NAME} \
    --name aws-load-balancer-controller \
    --namespace kube-system \
    --wait

# delete aws IAM policy
aws iam delete-policy \
    --policy-arn arn:aws:iam::${ACCOUNT_ID}:policy/AWSLoadBalancerControllerIAMPolicy


# delete cluster and other related AWS resources
eksctl delete cluster -f eks-cluster.yaml    
```