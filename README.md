# 🚀 EKS 3-Tier Application Deployment (Free-Tier Optimized)

## 📌 Project Overview

This project demonstrates the complete deployment of a **3-tier
application** (Frontend, Backend, and MySQL) on **Amazon EKS**,
optimized for small instance types (`t3.small`).

It includes:

-   EKS cluster provisioning
-   Persistent MySQL storage using EBS
-   LoadBalancer-based public access
-   Resource optimization
-   Full cleanup and cost verification process

------------------------------------------------------------------------

# 🏗 Architecture

    User
      │
      ▼
    AWS Elastic Load Balancer
      │
      ▼
    Frontend (Deployment)
      │
      ▼
    Backend (Deployment)
      │
      ▼
    MySQL (StatefulSet + EBS Volume)

------------------------------------------------------------------------

# 🖥 Environment Setup (Ubuntu Server)

## 1️⃣ Install AWS CLI

``` bash
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
sudo apt install unzip -y
unzip awscliv2.zip
sudo ./aws/install
aws configure
```

## 2️⃣ Install kubectl

``` bash
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x kubectl
sudo mv kubectl /usr/local/bin/
```

## 3️⃣ Install eksctl

``` bash
curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
sudo mv /tmp/eksctl /usr/local/bin
```

------------------------------------------------------------------------

# ☸️ EKS Cluster Setup

## Delete Old Cluster (If Any)

``` bash
eksctl delete cluster   --name three-tier-cluster   --region ap-south-1
```

## Create New Cluster

``` bash
eksctl create cluster   --name three-tier-cluster   --region ap-south-1   --nodegroup-name workers   --node-type t3.small   --nodes 2   --managed
```

## Verify Pod Capacity

``` bash
kubectl describe node | grep -i pods -A5
```

Expected:

    pods: 11

------------------------------------------------------------------------

# 💾 Install EBS CSI Driver

## Associate OIDC Provider

``` bash
eksctl utils associate-iam-oidc-provider   --region ap-south-1   --cluster three-tier-cluster   --approve
```

## Install EBS CSI Add-on

``` bash
eksctl create addon   --name aws-ebs-csi-driver   --cluster three-tier-cluster   --region ap-south-1
```

## Verify Installation

``` bash
kubectl get pods -n kube-system | grep ebs
```

------------------------------------------------------------------------

# 📦 Application Deployment

``` bash
kubectl apply -f full.yml
kubectl get pods -n prod
kubectl get pvc -n prod
kubectl get svc -n prod
```

------------------------------------------------------------------------

# 🌐 Access Application

``` bash
kubectl get svc -n prod
```

Copy the `EXTERNAL-IP` and access:

    http://<EXTERNAL-DNS>

------------------------------------------------------------------------

# ⚙ Resource Optimization (t3.small)

## MySQL

``` yaml
requests:
  cpu: 100m
  memory: 256Mi
limits:
  cpu: 300m
  memory: 512Mi
```

## Backend

``` yaml
requests:
  cpu: 50m
  memory: 128Mi
limits:
  cpu: 200m
  memory: 256Mi
```

## Frontend

``` yaml
requests:
  cpu: 25m
  memory: 64Mi
limits:
  cpu: 150m
  memory: 128Mi
```

------------------------------------------------------------------------

# 🛠 Issues & Resolutions

-   **Too many pods** → Switched to `t3.small`
-   **ImagePullBackOff** → Used valid Docker image tag
-   **EBS CSI stack conflict** → Deleted existing CloudFormation stack
-   **Termination protection enabled** → Disabled before stack deletion

------------------------------------------------------------------------

# 🧹 Cleanup (Avoid AWS Charges)

``` bash
eksctl delete cluster   --name three-tier-cluster   --region ap-south-1
```

------------------------------------------------------------------------

# 🧾 AWS Cleanup Verification Guide

## 1️⃣ Verify EKS Clusters

``` bash
aws eks list-clusters --region ap-south-1
```

Expected:

``` json
{
  "clusters": []
}
```

## 2️⃣ Verify No EC2 Instances

``` bash
aws ec2 describe-instances   --region ap-south-1   --query 'Reservations[*].Instances[*].[InstanceId,State.Name,InstanceType]'   --output table
```

## 3️⃣ Verify No Load Balancers

``` bash
aws elb describe-load-balancers --region ap-south-1
aws elbv2 describe-load-balancers --region ap-south-1
```

## 4️⃣ Verify No EBS Volumes

``` bash
aws ec2 describe-volumes   --region ap-south-1   --query 'Volumes[*].[VolumeId,State,Size]'   --output table
```

## 5️⃣ Verify No CloudFormation Stacks

``` bash
aws cloudformation list-stacks   --region ap-south-1   --query "StackSummaries[?StackStatus!='DELETE_COMPLETE'].[StackName,StackStatus]"   --output table
```

If termination protection is enabled:

``` bash
aws cloudformation update-termination-protection   --stack-name <stack-name>   --no-enable-termination-protection   --region ap-south-1

aws cloudformation delete-stack   --stack-name <stack-name>   --region ap-south-1
```

## 6️⃣ Verify No EKS Security Groups

``` bash
aws ec2 describe-security-groups   --region ap-south-1   --query 'SecurityGroups[*].[GroupId,GroupName]'   --output table
```

------------------------------------------------------------------------

# ✅ Safe-to-Close Checklist

You are safe from AWS charges if:

-   No EKS clusters exist
-   No EC2 instances are running
-   No Load Balancers exist
-   No EBS volumes remain
-   No CloudFormation stacks exist
-   No EKS-related security groups remain

------------------------------------------------------------------------

# 🎯 Final Outcome

Successfully deployed a production-style **3-tier application on Amazon
EKS** with:

-   Persistent MySQL storage using EBS
-   Scalable frontend & backend deployments
-   Public LoadBalancer access
-   Optimized resource configuration
-   Clean teardown procedure
