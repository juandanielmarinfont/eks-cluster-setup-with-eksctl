# EKS Cluster Setup with eksctl

## Project Description
This repository provides step-by-step instructions for setting up an Amazon EKS (Elastic Kubernetes Service) cluster using `eksctl`. The goal is to create a production-ready Kubernetes infrastructure that follows AWS best practices, allowing you to deploy scalable and resilient applications in the cloud.

### Prerequisites

Before starting, ensure you have the following tools installed and configured:

#### ⚙️ 1. Install AWS CLI

To interact with AWS services, you'll need to install the AWS Command Line Interface (CLI):

```bash
# Download and install AWS CLI
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
```

#### ⚙️ 2. Configure AWS CLI

Once you have the AWS CLI installed, configure it with your AWS credentials:

```bash
aws configure
```

You will be prompted for:

* **AWS Access Key ID**: Your access key.
* **AWS Secret Access Key**: The secret key associated with your access key.
* **Default region name**: The region where you want to deploy the cluster (e.g., us-east-1).
* **Default output format**: Typically json.

#### ⚙️ 3. Install eksctl

`eksctl` is a simple command-line tool for creating and managing EKS clusters.

```bash
# Download and move the binary
curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
sudo mv /tmp/eksctl /usr/local/bin

# Verify installation
eksctl version
```

#### ⚙️ 4. Install kubectl

`kubectl` is the command-line tool used to interact with Kubernetes clusters. To install kubectl, run the following command:

Install kubectl on Linux:

```bash
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl

# Verify the installation
kubectl version --client
```

### Step-by-Step Cluster Creation

#### 🚀 1. Export Environment Variables

Set the environment variables for the cluster configuration. Replace the placeholders with your own values.

```bash
export CLUSTER_NAME=<cluster-name>         # e.g., "eks-app-cluster"
export REGION=<region>                     # e.g., "us-east-1"
export NODE_GROUP_NAME=<node-group-name>   # e.g., "eks-app-nodes"
export NODE_TYPE=<instance-type>           # e.g., "t3.medium"
export NODES=<desired-number-of-nodes>     # e.g., "3"
export NODES_MIN=<min-number-of-nodes>     # e.g., "2"
export NODES_MAX=<max-number-of-nodes>     # e.g., "5"
```

#### 🚀 2. Create the EKS Cluster

Using `eksctl`, you can create an EKS cluster with the following command:

```bash
eksctl create cluster \
  --name $CLUSTER_NAME \
  --region $REGION \
  --nodegroup-name $NODE_GROUP_NAME \
  --node-type $NODE_TYPE \
  --nodes $NODES \
  --nodes-min $NODES_MIN \
  --nodes-max $NODES_MAX \
  --managed \
  --with-oidc \
  --enable-logging
```

#### 🔧 3. Configure kubectl to Connect to the Cluster

After the EKS cluster is created, you need to configure `kubectl` to interact with the new cluster:

```bash
aws eks --region $REGION update-kubeconfig --name $CLUSTER_NAME
```

To verify that the cluster is correctly configured and `kubectl` is connected, you can check the status of the nodes:

```bash
kubectl get nodes
```

#### 🗑️ 4. Deleting the EKS Cluster

If you need to clean up and remove the cluster, you can use the following command:

```bash
eksctl delete cluster --name $CLUSTER_NAME
```

This will delete the EKS cluster, including all associated resources.

### Best Practices

* **Node Auto-scaling**: Ensure the cluster can auto-scale by setting minimum and maximum nodes for the worker node group (`--nodes-min` and `--nodes-max` flags). 
* **IAM Roles**: Make sure your IAM roles are correctly configured, with permissions for managing EKS, EC2, and other AWS services. 
* **Security Groups and Networking**: Use VPC, subnets, and security groups following AWS best practices for networking and security. 
* **Monitoring and Logging**: Consider enabling CloudWatch logging for EKS control plane and nodes to monitor the health of the cluster.

### Troubleshooting 

* **Issue**: `kubectl` cannot connect to the cluster. 
    * **Solution**: Ensure that your AWS CLI is correctly configured, and that `kubectl` is using the correct kubeconfig. Run `aws eks update-kubeconfig` again if necessary. 
* **Issue**: Cluster creation is stuck. 
    * **Solution**: Check AWS CloudFormation for stack creation status and troubleshoot any failed resources.

### License

This project is licensed under the [MIT License](./LICENSE).