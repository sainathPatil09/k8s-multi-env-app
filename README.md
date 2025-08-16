# **Multi Environment EKS Cluster with Monitoring**

## **Project Overview:**

This project demonstrates a robust Kubernetes setup on AWS EKS with multi-node clusters and multi-environment namespaces. It implements best practices like dynamic volume provisioning, rolling updates, and cluster monitoring.

## **Features:**
- **AWS EKS Multi-Node Cluster:** Scalable and highly available Kubernetes cluster on AWS.
- **Multi-Namespace Environment:** Separate namespaces for **`dev`**, **`staging`**, and **`prod`** environments.
- **MongoDB with EBS Volumes:** Persistent storage using AWS EBS for MongoDB data.
- **Dynamic Volume Provisioning:** Persistent Volumes created on-demand, waiting for consumer pods.
- **Rolling Updates Deployment Strategy:** Zero-downtime updates for deployments.
- **Cluster Monitoring:** Integrated monitoring with **Prometheus** and **Grafana** for metrics and alerts.

## **Architecture**


## **Prerequisites**

- AWS Account

*I will help you to setup this cluster even if you are a beginner*

## **Setup Instruction:**

### **Step 1 Lunch Bastion Host:**

- What is Bastion Host ?
  - The Server through we will control our Cluster, This EC2 will not be the part of our cluster
- Lunch `T2.micro`
- Update EC2  `sudo apt update`

### **Step 2 Install AWS CLI and Configure:**
- Install AWS CLI command
```
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
sudo apt install unzip
unzip awscliv2.zip
sudo ./aws/install
```

- Configure AWS 
    - Go to AWS IAM and create User
    - create `Access Key` and `Secret Access Key`
    - Then use this command `AWS configure`
    - it will ask for your accesskey and sceret accesskey, copy past them
    - now your Bastion Host is configured with your AWS

### **Step 3 Install kubectl ,esksctl and Helm:**

- kubectl install command:
```
curl -o kubectl https://amazon-eks.s3.us-west-2.amazonaws.com/1.19.6/2021-01-05/bin/linux/amd64/kubectl
chmod +x ./kubectl
sudo mv ./kubectl /usr/local/bin
kubectl version --short --client
```
- eksctl install command
```
curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
sudo mv /tmp/eksctl /usr/local/bin
eksctl version
```
- Helm install command
```
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
chmod 700 get_helm.sh
./get_helm.sh
```

### **Step 4 Create Cluster:**
- Create EKS Cluster
```bash
eksctl create cluster --name=my-cluster \
                      --region=ap-south-1 \
                      --version=1.30 \
                      --without-nodegroup
```

- Associate IAM OIDC Provider
```bash
eksctl utils associate-iam-oidc-provider \
    --region ap-south-1 \
    --cluster my-cluster \
    --approve
```
#

- Create Nodegroup
```bash
eksctl create nodegroup --cluster=my-cluster \
                       --region=ap-south-1 \
                       --name=my-cluster \
                       --node-type=t2.medium \
                       --nodes=2 \
                       --nodes-min=2 \
                       --nodes-max=2 \
                       --node-volume-size=25 \
                       --ssh-access \
                       --ssh-public-key=ec2_keypair 
```
#### Note: Make sure the ssh-public-key "ec2_keypair is available in your aws account"
#

- Update Kubectl Context
```bash
aws eks update-kubeconfig --region ap-south-1 --name my-cluster
```
#

- Delete EKS Cluster
```bash
eksctl delete cluster --name=my-cluster --region=ap-south-1
```

### **Step 5 Install AWS EBS CSI Driver:**
- Add the AWS EBS CSI Driver Helm chart repository:
```
helm repo add aws-ebs-csi-driver https://kubernetes-sigs.github.io/aws-ebs-csi-driver
helm repo update
```
- Identify Node Group IAM Role:
```
aws eks describe-nodegroup \
  --cluster-name <your-cluster-name> \
  --nodegroup-name <your-nodegroup-name> \
  --query "nodegroup.nodeRole" --output text

```





