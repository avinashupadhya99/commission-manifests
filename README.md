# Manifests for deploying `Commission` application on EKS

```bash
eksctl create cluster -f eks-cluster-config.yaml
```

```bash
alias k=kubectl
```

```bash
k apply -f order-service/database-secret.yaml
k apply -f order-service/deployment.yaml
```

https://docs.aws.amazon.com/eks/latest/userguide/aws-load-balancer-controller.html

```bash
aws eks describe-cluster --name commission-cluster --query "cluster.identity.oidc.issuer" --output text
```

```bash
aws iam list-open-id-connect-providers | grep XXXXXXXXXXXXXXXXXXXXX
```

```bash
eksctl utils associate-iam-oidc-provider --cluster commission-cluster --approve
```


```bash
aws iam create-policy \
    --policy-name AWSLoadBalancerControllerIAMPolicy \
    --policy-document file://iam_policy.json
```

```bash
eksctl create iamserviceaccount \
  --cluster=commission-cluster \
  --namespace=kube-system \
  --name=aws-load-balancer-controller \
  --role-name "AmazonEKSLoadBalancerControllerRole" \
  --attach-policy-arn=arn:aws:iam::<account_id>:policy/AWSLoadBalancerControllerIAMPolicy \
  --approve
```

```bash
helm install aws-load-balancer-controller eks/aws-load-balancer-controller \
  -n kube-system \
  --set clusterName=commission-cluster \
  --set serviceAccount.create=false \
  --set serviceAccount.name=aws-load-balancer-controller \
  --set image.repository=602401143452.dkr.ecr.ap-south-1.amazonaws.com/amazon/aws-load-balancer-controller \
  --set region=ap-south-1 \
  --set vpcId=vpc-0e3c93c8d7fcc79fa
```

https://aws.amazon.com/premiumsupport/knowledge-center/eks-set-up-externaldns/

```bash
aws iam create-policy \
    --policy-name Route53ExternalDNSIAMPolicy \
    --policy-document file://route53_iam_policy.json
```

```bash
eksctl create iamserviceaccount \
  --name external-dns \
  --namespace kube-system \
  --cluster commission-cluster  \
  --attach-policy-arn arn:aws:iam::<account_id>:policy/Route53ExternalDNSIAMPolicy \
  --approve
```

```bash
k apply -f external-dns.yaml
``` 

```bash
k apply -f order-service/service.yaml
k apply -f order-service/ingress.yaml
```