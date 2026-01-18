# EKS-with-K8s
 
Prerequest installation for intract with AWS EKS
-------------------------------------------------
prerequisites-
kubectl-
------- 
 A command line tool for working with Kubernetes clusters. For more information, see Installing or updating kubectl.

-curl -LO https://dl.k8s.io/release/$(curl -Ls https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl
-chmod +x kubectl
-sudo mv kubectl /usr/local/bin/
-kubectl version --client->installed succsesfully


eksctl- 
------ 
A command line tool for working with EKS clusters that automates many individual tasks. For more information, see Installing or updating.
-curl --silent --location "https://github.com/eksctl-io/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
-sudo mv /tmp/eksctl /usr/local/bin
-eksctl version
-eksctl get cluster


AWS CLI –
--------
 A command line tool for working with AWS services, including Amazon EKS. For more information, see Installing, updating, and uninstalling the AWS CLI in the AWS Command Line Interface User Guide. After installing the AWS CLI, we recommend that you also configure it. For more information, see Quick configuration with aws configure in the AWS Command Line Interface User Guide.

-curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
-unzip awscliv2.zip
-sudo apt install unzip
-sudo ./aws/install
-aws --version

Configure CLI-
-------------
-aws configure
Acsses key -  
secreat key -  
-aws sts get-caller-identity -verify connected to AWS

create EKS cluster using CLI-
----------------------------
-eksctl create cluster --name demo-cluster --region us-east-1 --fargate  -> this for create control plane 
-eksctl delete cluster --name demo-cluster --region us-east-1  ->dont forgot to delete cluster if practical is done 
-aws eks update-kubeconfig --name demo-cluster --region us-east-1 ->It creates or updates your kubeconfig file so that kubectl can talk to your EKS cluster.

-Why do we need a Fargate profile?
------------------------------------

EKS supports two compute options:

✅ 1. EC2 nodes

You manage worker nodes (servers) yourself.

✅ 2. AWS Fargate

AWS runs the pods for you — serverless Kubernetes.

A Fargate profile tells EKS:

which namespaces/pods should use Fargate

which pods should run on EC2 nodes (if you have any)

-eksctl create fargateprofile \
    --cluster demo-cluster \
    --region us-east-1 \
    --name alb-sample-app \
    --namespace game-2048

-kubectl apply -f https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.5.4/docs/examples/2048/2048_full.yaml  -> this link copy from github project after this apply depoyment/ingress etc create

-eksctl utils associate-iam-oidc-provider --cluster $cluster_name --approve

🔥 Why this is needed
EKS uses OIDC for IRSA

When you want your Kubernetes pods to access AWS services (S3, DynamoDB, etc.) without AWS keys, you use IRSA.

To use IRSA, your cluster must have an OIDC provider attached.

Above command does exactly that.

-curl -O https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.11.0/docs/install/iam_policy.json

It downloads the IAM policy JSON file used by the AWS Load Balancer Controller.

🔥 Why you need it

The AWS Load Balancer Controller needs AWS permissions to:

✅ create/load/manage ALBs
✅ create target groups
✅ modify security groups
✅ manage listeners
✅ etc.

That policy defines exactly what permissions the controller needs.

-aws iam create-policy \         ->create policy
    --policy-name AWSLoadBalancerControllerIAMPolicy \
    --policy-document file://iam_policy.json


eksctl create iamserviceaccount \   --> creat role
  --cluster=<your-cluster-name> \
  --namespace=kube-system \
  --name=aws-load-balancer-controller \
  --role-name AmazonEKSLoadBalancerControllerRole \
  --attach-policy-arn=arn:aws:iam::<your-aws-account-id>:policy/AWSLoadBalancerControllerIAMPolicy \
  --approve


AWS Load Balancer Controller using Helm.
-------------------------------------------
-curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash  ->install helm
-helm version
-helm repo add eks https://aws.github.io/eks-charts
-helm repo update

-helm install aws-load-balancer-controller eks/aws-load-balancer-controller -n kube-system \  -->install LB controller
  --set clusterName=<your-cluster-name> \
  --set serviceAccount.create=false \
  --set serviceAccount.name=aws-load-balancer-controller \
  --set region=<your-region> \
  --set vpcId=<your-vpc-id>

Verify that the deployments are running.

-kubectl get deployment -n kube-system aws-load-balancer-controller

Acsses aplication link-

-kubectl get ns
-kubectl get ingress -n game-2048

