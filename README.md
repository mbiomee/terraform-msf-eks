# terraform-eks-msf


[![TerraformRefigistry](https://img.shields.io/badge/Terraform%20Registry-v1.0.2-blue.svg)](https://registry.terraform.io/modules/mbiomee/eks/msf/)


General aws eks deploy using Terraform
---

##for more info
[Amazon Elastic Kubernetes Service](https://aws.amazon.com/eks/)
[Terraform](https://www.terraform.io/)

## Caution
applying this infrastructure will create resources on your aws account so be sure you have enough credits
to see resources [click here](https://registry.terraform.io/modules/mbiomee/eks/msf/?tab=resources)
  
##Prerequisites

1. aws account
2. IAM role with AdminstratorAccess policy
3. machine with aws cli, terraform cli, kubectl installed
4. create ssh key and upload the public key to your EC2 region


##Usage

create main.tf file and pass your variables link example
```terraform
module "eks" {
  source              = "mbiomee/eks/msf"
  aws-region          = "us-east-1"
  availability-zones  = ["us-east-1a", "us-east-1b", "us-east-1c"]
  cluster-name        = "msf-cluster"
  k8s-version         = "1.13"
  node-instance-type  = "t3.medium"
  root-block-size     = "40"
  desired-capacity    = "3"
  max-size            = "5"
  min-size            = "1"
  public-min-size     = "1"
  public-max-size     = "1"
  public-desired-capacity = "1"
  vpc-subnet-cidr     = "10.0.0.0/16"
  private-subnet-cidr = ["10.0.0.0/19", "10.0.32.0/19", "10.0.64.0/19"]
  public-subnet-cidr  = ["10.0.128.0/20", "10.0.144.0/20", "10.0.160.0/20"]
  db-subnet-cidr      = ["10.0.192.0/21", "10.0.200.0/21", "10.0.208.0/21"]
  eks-cw-logging      = ["api", "audit", "authenticator", "controllerManager", "scheduler"]
  ec2-key             = "my-key"
}

output "kubeconfig" {
  value = module.eks.kubeconfig
}

output "config-map" {
  value = module.eks.config-map
}

```

**Or** by using variables.tf or a tfvars file:

```terraform
module "eks" {
  source              = "mbiomee/eks/msf"
  aws-region          = var.aws-region
  availability-zones  = var.availability-zones
  cluster-name        = var.cluster-name
  k8s-version         = var.k8s-version
  node-instance-type  = var.node-instance-type
  root-block-size     = var.root-block-size
  desired-capacity    = var.desired-capacity
  max-size            = var.max-size
  min-size            = var.min-size
  public-min-size     = var.public-min-size
  public-max-size     = var.public-max-size
  public-desired-capacity = var.public-desired-capacity
  vpc-subnet-cidr     = var.vpc-subnet-cidr
  private-subnet-cidr = var.private-subnet-cidr
  public-subnet-cidr  = var.public-subnet-cidr
  db-subnet-cidr      = var.db-subnet-cidr
  eks-cw-logging      = var.eks-cw-logging
  ec2-key             = var.ec2-key
}
```

### Terraform

You need to run the following commands to create the resources with Terraform:

```bash
terraform init
terraform plan
terraform apply
```



### Setup kubectl

Setup your `KUBECONFIG`

```bash
terraform output kubeconfig > ~/.kube/eks-cluster
export KUBECONFIG=~/.kube/eks-cluster
```

### Authorize worker nodes

Get the config from terraform output, and save it to a yaml file:

```bash
terraform output config-map > config-map-aws-auth.yaml
```

Apply the config map to EKS:

```bash
kubectl apply -f config-map-aws-auth.yaml
```

You can verify the worker nodes are joining the cluster

```bash
kubectl get nodes --watch
```

### Cleaning up

You can destroy this cluster entirely by running:

```bash
terraform plan -destroy
terraform destroy  --force
```
