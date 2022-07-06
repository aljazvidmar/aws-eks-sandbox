# Example application deployment on Kubernetes (EKS) on AWS with Nginx Ingress controller

The deployment consists of the following:
- crating a new AWS Elastic Kubernetes Services (EKS) cluster with `eksctl`
- provisioning a Network Load Balancer (NLB) with Nginx ingress controller
- creating TLS termination on NLB load balancer with an AWS provided wildcard certificate
- deployment of Cheese APP (https://hub.docker.com/r/errm/cheese)


## Files

- `eks-cluster.yaml` is a EKS cluster definition for `eksctl`
- `cheese-deploymens.yaml`, `cheese-services.yaml`, `cheese-ingress.yaml` are k8 deployment files for app, services an ingress
- `nlb-with-tls-termination-template.yaml` - official deployment file for nginx-ingress wich will be modified with real VPC IP andcert ID
- `deployment.md` and `cleanup.md` instructions for deploying and cleaning up 



## Deployment of Nginx Ingress Controller

```bash
export CLUSTER_NAME="k8-nlb-cluster"
export ACCOUNT_ID=$(aws sts get-caller-identity --query Account --output text)
export WILDCARD_DOMAIN='*.adesignstudio.net' # your domain here

# use cluster_name variable inside eksctl cluster config yaml
cat eks-cluster-template.yaml | envsubst '${CLUSTER_NAME}' > eks-cluster.yaml

# create cluster
eksctl create cluster -f eks-cluster.yaml

# get vpc cidr and aws certificate id
export eks_cluster_vpc_cidr=$(aws ec2 describe-vpcs --vpc-ids $(aws eks describe-cluster --name $CLUSTER_NAME --query 'cluster.resourcesVpcConfig.vpcId' --output text) --query 'Vpcs[*].CidrBlockAssociationSet[*].CidrBlock' --output text)
export aws_certificate=$(aws acm list-certificates --query "CertificateSummaryList[?DomainName=='$WILDCARD_DOMAIN'].CertificateArn" --output text)

# create an actual deployment for nginx-ingress with TLS configured with actual EKS Cluster VPS CIRD and certificate ID
cp -f nlb-with-tls-termination-template.yaml nlb-with-tls-termination.yaml

sed -i "s%XXX.XXX.XXX/XX%$eks_cluster_vpc_cidr%g" nlb-with-tls-termination.yaml
sed -i "s%arn:aws:acm:us-west-2:XXXXXXXX:certificate/XXXXXX-XXXXXXX-XXXXXXX-XXXXXXXX%$aws_certificate%g" nlb-with-tls-termination.yaml

# deploy nginx-ingress controller and nlb
kubectl apply -f nlb-with-tls-termination.yaml
```


## Deployment of cheese demo app

```bash
# deploy cheese app
kubectl apply -f cheese-deployments.yaml
kubectl apply -f cheese-services.yaml
kubectl apply -f cheese-ingress.yaml
```

### Load balancer provisioning

After one the load balancer is provisioned, which takes about a few minutes, you have to assign its DNS name to a new CNAME record in DNS domain managment. This allows that the domain (e.g. `cheese.example.com`) reaches the AWS load balancer.
