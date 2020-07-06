# EKS_TERRAFORM

**What is Terraform?**

Terraform is an open-source “Infrastructure as Code” tool, created by HashiCorp. A declarative coding tool, Terraform enables developers to use a high-level configuration language called HCL (HashiCorp Configuration Language) to describe the desired “end-state” cloud or on-premises infrastructure for running an application. It then generates a plan for reaching that end-state and executes the plan to provision the infrastructure.

**What is Kubernetes?**

Kubernetes (also known as k8s or “Kube”) is an open-source container orchestration platform that automates many of the manual processes involved in deploying, managing and scaling containerized applications.

**What is EKS?**

Amazon Elastic Kubernetes Service (Amazon EKS) is a fully managed Kubernetes service. Simply put, EKS is a managed containers-as-a-service (CaaS).

**Prerequisite**

Should have an AWS account.
Terraform should be installed in your system.
Now, let's move to our main objective:

I have divide whole Terraform code into two parts:

**Part1**

In this part we have to specify that which provider we want to use also we have to specify region in which we want our cluster to get deployed.

```provider "aws" {
  region     = "ap-south-1"
  profile    = "default"
}
```
**Part2**

Here we have to create all the resources that we will need to deploy the EKS cluster. In this part, Terraform will create IAM Role and two policies which will be associated with the EKS cluster.

```resource "aws_iam_role" "ekscluster1" {
  name = "ekscluster1"

  assume_role_policy = <<POLICY
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "eks.amazonaws.com"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
POLICY
}

resource "aws_iam_role_policy_attachment" "AmazonEKSClusterPolicy" {
  policy_arn = "arn:aws:iam::aws:policy/AmazonEKSClusterPolicy"
  role       = aws_iam_role.ekscluster1.name
}

resource "aws_iam_role_policy_attachment" "AmazonEKSServicePolicy" {
  policy_arn = "arn:aws:iam::aws:policy/AmazonEKSServicePolicy"
  role       = aws_iam_role.ekscluster1.name
}
```
In this snippet, we will be creating the EKS cluster. Subnet id of vpc should be known as this EKS cluster will be launched in default VPC.

```resource "aws_eks_cluster" "aws_eks1" {
  name     = "udit_eks_cluster"
  role_arn = aws_iam_role.ekscluster1.arn

  vpc_config {
    subnet_ids = ["subnet-2c7e1560","subnet-430ebc38"]
  }

  tags = {
    Name = "EKS_clust"
  }
}
```

Now, we will create three policies for NODE GROUPS. We have to also define Scaling Configuration Block.

***desired_size - Desired number of worker nodes.
max_size - Maximum number of worker nodes.
min_size - Minimum number of worker nodes.***


```resource "aws_iam_role" "eks_nodes" {
  name = "eks_node_group"

  assume_role_policy = <<POLICY
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "ec2.amazonaws.com"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
POLICY
}

resource "aws_iam_role_policy_attachment" "AmazonEKSWorkerNodePolicy" {
  policy_arn = "arn:aws:iam::aws:policy/AmazonEKSWorkerNodePolicy"
  role       = aws_iam_role.eks_nodes.name
}

resource "aws_iam_role_policy_attachment" "AmazonEKS_CNI_Policy" {
  policy_arn = "arn:aws:iam::aws:policy/AmazonEKS_CNI_Policy"
  role       = aws_iam_role.eks_nodes.name
}

resource "aws_iam_role_policy_attachment" "AmazonEC2ContainerRegistryReadOnly" {
  policy_arn = "arn:aws:iam::aws:policy/AmazonEC2ContainerRegistryReadOnly"
  role       = aws_iam_role.eks_nodes.name
}

resource "aws_eks_node_group" "node" {
  cluster_name    = aws_eks_cluster.aws_eks1.name
  node_group_name = "ng1"
  node_role_arn   = aws_iam_role.eks_nodes.arn
  subnet_ids      = ["subnet-2c7e1560","subnet-430ebc38"]

  scaling_config {
    desired_size = 1
    max_size     = 1
    min_size     = 1
  }

  depends_on = [
    aws_iam_role_policy_attachment.AmazonEKSWorkerNodePolicy,
    aws_iam_role_policy_attachment.AmazonEKS_CNI_Policy,
    aws_iam_role_policy_attachment.AmazonEC2ContainerRegistryReadOnly,
  ]
}
```
To deploy the EKS cluster we have to run this file.

**Commands Used**

***terraform init - This will download all the required plugins or will set up all the configuration files that are required for running the file.***
***terraform apply - This command will run the file. Terraform will describe each and every resource that will be created.***
***terraform destroy - This command will delete all the resources that were created using this file.***
