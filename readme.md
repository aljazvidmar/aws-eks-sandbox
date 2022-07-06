# This is a sandbox for AWS Elastic Kubernetes Service

    
## Prerequisites

1. AWS login with IAM credentials
2. Existing wildcard certificate on AWS (`*.example.com`)
3. terminal environment (for example: WIN10 + Git Bash)
4. Installed:
    - kubectl (https://kubernetes.io/docs/tasks/tools/install-kubectl-windows/)
    - awscli (https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html)
    - eksctl (https://docs.aws.amazon.com/eks/latest/userguide/eksctl.html)
5. Configured awscli (https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-envvars.html)
6. Access to domain managment to add a `CNAME` record

---

# Contents
- [AWS ALB Ingress Controller setup](aws_alb_ingress/deployment.md)
- [AWS NLB Nginx Ingress Controller setup](aws_nginx_ingress/deployment.md)
- [Monitoring with Prometheus and Grafana](monitoring_with_prometheus_and_grafana/deployment.md)




