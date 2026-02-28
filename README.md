EKS 3-Tier Application Deployment Documentation
1. Project Overview
This document describes the complete setup and deployment of a 3-tier application (Frontend, Backend, and MySQL) on Amazon EKS using optimized configuration suitable for small instance types (t3.small).

2. Architecture
User → AWS Elastic Load Balancer → Frontend (Deployment) → Backend (Deployment) → MySQL (StatefulSet + EBS Volume)

3. Environment Setup (Ubuntu Server)
3.1 Install AWS CLI
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
sudo apt install unzip -y
unzip awscliv2.zip
sudo ./aws/install
aws configure
3.2 Install kubectl
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x kubectl
sudo mv kubectl /usr/local/bin/
3.3 Install eksctl
curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
sudo mv /tmp/eksctl /usr/local/bin

4. EKS Cluster Setup
4.1 Delete Old Cluster (if any)
eksctl delete cluster --name three-tier-cluster --region ap-south-1
4.2 Create New Cluster
eksctl create cluster \
  --name three-tier-cluster \
  --region ap-south-1 \
  --nodegroup-name workers \
  --node-type t3.small \
  --nodes 2 \
  --managed
4.3 Verify Pod Capacity
kubectl describe node | grep -i pods -A5
Expected: pods: 11

5. Install EBS CSI Driver
5.1 Associate OIDC Provider
eksctl utils associate-iam-oidc-provider \
  --region ap-south-1 \
  --cluster three-tier-cluster \
  --approve
5.2 Install EBS CSI Addon
eksctl create addon \
  --name aws-ebs-csi-driver \
  --cluster three-tier-cluster \
  --region ap-south-1
5.3 Verify EBS CSI
kubectl get pods -n kube-system | grep ebs
Expected: ebs-csi-controller and ebs-csi-node in Running state

6. Application Deployment
kubectl apply -f full.yml
kubectl get pods -n prod
kubectl get pvc -n prod
kubectl get svc -n prod

7. Resource Optimization
MySQL:
  requests: cpu 100m, memory 256Mi
  limits: cpu 300m, memory 512Mi

Backend:
  requests: cpu 50m, memory 128Mi
  limits: cpu 200m, memory 256Mi

Frontend:
  requests: cpu 25m, memory 64Mi
  limits: cpu 150m, memory 128Mi

8. Troubleshooting Encountered
- Too many pods error (resolved by using t3.small)
- ImagePullBackOff (resolved by using correct Docker image tag)
- EBS CSI stack conflict (resolved by deleting existing CloudFormation stack)

9. Access Application
kubectl get svc -n prod
Copy LoadBalancer EXTERNAL-IP and access via browser.

10. Cleanup to Avoid Charges
eksctl delete cluster --name three-tier-cluster --region ap-south-1

11. Final Outcome
Successfully deployed a production-style 3-tier application on Amazon EKS with:
- Persistent MySQL storage using EBS
- Scalable frontend and backend deployments
- Public LoadBalancer access
- Optimized resource configuration
- Clean teardown procedure
