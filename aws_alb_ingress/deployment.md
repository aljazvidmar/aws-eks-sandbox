# Example application deployment on Kubernetes (EKS) on AWS with ALB controller

The deployment consists of the following:
- crating a new AWS Elastic Kubernetes Services (EKS) cluster with `eksctl`
- provisioning a Application Load Balancer (ALB) with ALB ingress controller
- creating TLS termination on ALB load balancer with an AWS provided wildcard certificate
- deployment of Cheese APP (https://hub.docker.com/r/errm/cheese)


## Files

- `eks-cluster.yaml` is a EKS cluster definition for `eksctl`
- `cheese-deploymens.yaml`, `cheese-services.yaml`, `cheese-ingress.yaml` are k8 deployment files for app, services an ingress
- `deployment.md` and `cleanup.md` instructions for deploying and cleaning up 



## Deployment of AWS Application Load Balancer (ALB) Controller

```bash
# export needed variables
export LBC_VERSION="v2.4.1"
export LBC_CHART_VERSION="1.4.1"
export AWS_REGION="us-east-1"
export CLUSTER_NAME="my-k8cluster"
export ACCOUNT_ID=$(aws sts get-caller-identity --query Account --output text)


# use cluster_name variable inside eksctl cluster config yaml
cat eks-cluster-template.yaml | envsubst '${CLUSTER_NAME}' > eks-cluster.yaml


# create cluster
eksctl create cluster -f eks-cluster.yaml


# Create IAM OIDC provider
eksctl utils associate-iam-oidc-provider \
    --region ${AWS_REGION} \
    --cluster ${CLUSTER_NAME} \
    --approve


# Create an IAM policy
curl -o iam_policy.json https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/${LBC_VERSION}/docs/install/iam_policy.json
aws iam create-policy \
    --policy-name AWSLoadBalancerControllerIAMPolicy \
    --policy-document file://iam_policy.json


# Create a IAM role and ServiceAccount
eksctl create iamserviceaccount \
  --cluster ${CLUSTER_NAME} \
  --namespace kube-system \
  --name aws-load-balancer-controller \
  --attach-policy-arn arn:aws:iam::${ACCOUNT_ID}:policy/AWSLoadBalancerControllerIAMPolicy \
  --override-existing-serviceaccounts \
  --approve    


# Install the TargetGroupBinding CRDs
kubectl apply -k "github.com/aws/eks-charts/stable/aws-load-balancer-controller/crds?ref=master"


# deploy aws load balancer controller with helm
helm repo add eks https://aws.github.io/eks-charts

helm upgrade -i aws-load-balancer-controller \
    eks/aws-load-balancer-controller \
    -n kube-system \
    --set clusterName=${CLUSTER_NAME} \
    --set serviceAccount.create=false \
    --set serviceAccount.name=aws-load-balancer-controller \
    --set image.tag="${LBC_VERSION}" \
    --version="${LBC_CHART_VERSION}"

kubectl -n kube-system rollout status deployment aws-load-balancer-controller
```

## Deployment of cheese demo app

```bash
# deploy cheese app
kubectl apply -f cheese-deployments.yaml
kubectl apply -f cheese-services.yaml
kubectl apply -f cheese-ingress.yaml
```

wait the AWS ALB  to be provisioned and then add CNAME in dns managment for each cheese:
- `stilton.<your_domain>`
- `cheddar.<your_domain>`
- `wensleydale.<your_domain>`

