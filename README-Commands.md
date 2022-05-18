EKS Fargate Cluster Creation
-----------------------------
eksctl create cluster \
 --name "cluster-name" \
 --version "eks-cluster-version" \
 --region "aws-region-code" \
 --without-nodegroup \
 --with-oidc \
 --fargate \
 --profile "your-aws-account-profile"


eksctl create cluster --name my-eks-cluster --version 1.22 --region eu-west-2 --without-nodegroup --with-oidc --fargate --profile AWS_741032333307_User


Create IAM Policy
------------------
aws iam create-policy \
--policy-nam "policy-name" \
--policy-document "json-file-path" \
--profile "your-aws-account-profile"

aws iam create-policy --policy-name AWSLoadBalancerControllerIAMPolicy  --policy-document file://iam_policy.json --profile AWS_741032333307_User


Create Service Account
-----------------------
eksctl create iamserviceaccount  \
--cluster="cluster-name" \
--region "aws-region-code" \
--namespace="kubernetes-namespace" \
--name="aws-load-balancer-controller-name" \
--attach-policy-arn="policy-arn-of-aws"  \
--override-existing-serviceaccounts \
--approve \
--profile "your-aws-account-profile"


eksctl create iamserviceaccount  --cluster=my-eks-cluster  --region eu-west-2  --namespace=kube-system   --name=aws-load-balancer-controller  --attach-policy-arn=arn:aws:iam::741032333307:policy/AWSLoadBalancerControllerIAMPolicy  --override-existing-serviceaccounts --approve --profile AWS_741032333307_User	


Create AWS Load Balancer Controller using HELM
----------------------------------------------
helm install aws-load-balancer-controller eks/aws-load-balancer-controller \
-n "kubernetes-namespace" \
--set clusterName="cluster-name" \
--set region="aws-region-code>"\
--set vpcId="vpc-of-the-eks-cluster" \
--set serviceAccount.create=false \
--set serviceAccount.name="aws-load-balancer-controller-name" \
--set image.repository="docker-registry-image-like-of-aws-load-balancer-controller" \
--set profile="your-aws-account-profile"


helm install aws-load-balancer-controller eks/aws-load-balancer-controller  -n kube-system  --set clusterName=my-eks-cluster --set region=eu-west-2 --set vpcId=vpc-093e1abe881a58263 --set serviceAccount.create=false --set serviceAccount.name=aws-load-balancer-controller  --set image.repository=602401143452.dkr.ecr.eu-west-2.amazonaws.com/amazon/aws-load-balancer-controller --set profile=AWS_741032333307_User


Create Fargate Profile for application deployment
---------------------------------------------------
eksctl create fargateprofile \
--cluster "cluster-name" \
--name "fargate-profile-name" \
--region "aws-region-code" \
--namespace "kubernetes-namespace" \
--profile "your-aws-account-profile"


eksctl create fargateprofile --cluster my-eks-cluster --name app-fargate-profile --region eu-west-2 --namespace app-namespace --profile AWS_741032333307_User


Deploy the Application
------------------------
kubectl apply -f game-app.yaml




Reference:
https://github.com/stacksimplify/aws-eks-kubernetes-masterclass/tree/master/08-NEW-ELB-Application-LoadBalancers/08-01-Load-Balancer-Controller-Install
https://docs.aws.amazon.com/eks/latest/userguide/aws-load-balancer-controller.html
https://docs.aws.amazon.com/eks/latest/userguide/alb-ingress.html