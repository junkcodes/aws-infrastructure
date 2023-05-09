# AWS Infrastructure
Developing new infrastructure from scratch for a microservice-based architecture product requires a well-planned approach. To ensure a successful outcome, it is essential to break down the work into essential steps. The following steps can guide you through the process,
* Outline Technogoly Stacks
* Setup & Configure Technogoly Stacks
* Provision using Terraform

## Outline Technogoly Stacks
Developing a microservice-based architecture requires careful consideration of the technology stacks and tools used to ensure the system is scalable, fault-tolerant, secure, and highly available. Here, we outline the technology stacks and tools that we recommend for building a microservice-based architecture,
### AWS EKS
AWS EKS provides a highly available, fault tolerant, scalable, load balanced, and self-healing infrastructure for running containerized applications. It supports popular container orchestration tools like Kubernetes, which can be used to deploy and manage microservices in a scalable and flexible way.    
 
### AWS ElastiCache
The AWS Elasticache service provides a flexible and scalable solution for caching microservices in the Elastic Kubernetes Service (EKS) on AWS. By utilizing Elasticache, microservices can improve their performance and reduce latency by caching frequently accessed data, which can greatly enhance the overall user experience. With Elasticache, microservices can seamlessly integrate with the EKS and leverage its benefits while enjoying high availability, fault tolerance, and automatic scalability.   

### AWS CloudWatch
The AWS CloudWatch service provides a comprehensive solution for monitoring, logging, alerting, and security management in AWS. It offers a range of features that enable users to monitor their infrastructure, collect and analyze log data, set up alarms and notifications, and maintain the security of their AWS resources. With CloudWatch, users can gain insight into their system performance, troubleshoot issues, and ensure the overall health and availability of their applications and services. 

### Vault
Vault provides a secure way to store and manage secrets, and EKS can be used as a deployment target for Vault. With Vault HA, you can have multiple instances of Vault running simultaneously, ensuring that secrets are available even if one of the instances goes down. Additionally, Vault can be integrated with EKS through Kubernetes secrets or AWS IAM roles, allowing for seamless authentication and authorization. Overall, Vault HA can provide a robust and secure solution for secrets and key management in EKS.      

### AWS IAM 
AWS Identity and Access Management (IAM) can be used to manage access to various AWS services, including Amazon Elastic Kubernetes Service (EKS), Amazon CloudWatch, and Amazon ElastiCache. IAM allows you to create and manage AWS users, groups, and roles, and define their permissions to access AWS resources.    

### Terraform
Terraform can be used to provision AWS EKS, IAM, CloudWatch, and ElasticCache. In fact, Terraform has built-in support for provisioning AWS resources, including EKS clusters, IAM roles and policies, CloudWatch alerts and logs, and ElasticCache clusters. This makes it easier for developers to manage and automate the infrastructure needed for their microservice-based architectures

## Setup & Configure Technogoly Stacks
### AWS EKS
[Getting started with Amazon EKS](https://docs.aws.amazon.com/eks/latest/userguide/getting-started.html)
### AWS ElastiCache
[Getting started with Amazon ElastiCache for Redis](https://docs.aws.amazon.com/AmazonElastiCache/latest/red-ug/GettingStarted.html)
### AWS CloudWatch
[Getting started with Amazon CloudWatch](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/GettingStarted.html)
### Vault
[Vault HA Guide](https://github.com/junkcodes/vault-ha-guide)
### AWS IAM
[Getting started with IAM](https://docs.aws.amazon.com/IAM/latest/UserGuide/getting-started.html)
### Terraform
[Terraform Getting Started - AWS](https://developer.hashicorp.com/terraform/tutorials/aws-get-started)

## Provision using Terraform (Except Vault)
With Terraform, we can declare desired infrastructure configuration as code, and then deploy and manage that infrastructure in a consistent manner. In this particular project, Terraform can be used to provision AWS EKS, IAM, CloudWatch, and ElasticCache resources. This approach ensures that the infrastructure is highly available, fault-tolerant, scalable, and secure, and that it can be easily managed and updated as needed. Here, we will outline the Terraform modules used to provision these AWS services,
### AWS EKS:
Terraform AWS EKS module by Terraform AWS modules [EKS](https://registry.terraform.io/modules/terraform-aws-modules/eks/aws/latest).
Here is an example script for provisioning EKS via terraform,
```
module "eks" {
  source  = "terraform-aws-modules/eks/aws"
  version = "~> 19.0"

  cluster_name    = "my-cluster"
  cluster_version = "1.24"

  cluster_endpoint_public_access  = true

  cluster_addons = {
    coredns = {
      most_recent = true
    }
    kube-proxy = {
      most_recent = true
    }
    vpc-cni = {
      most_recent = true
    }
  }

  vpc_id                   = "vpc-1234556abcdef"
  subnet_ids               = ["subnet-abcde012", "subnet-bcde012a", "subnet-fghi345a"]
  control_plane_subnet_ids = ["subnet-xyzde987", "subnet-slkjf456", "subnet-qeiru789"]

  # Self Managed Node Group(s)
  self_managed_node_group_defaults = {
    instance_type                          = "m6i.large"
    update_launch_template_default_version = true
    iam_role_additional_policies = {
      AmazonSSMManagedInstanceCore = "arn:aws:iam::aws:policy/AmazonSSMManagedInstanceCore"
    }
  }

  self_managed_node_groups = {
    one = {
      name         = "mixed-1"
      max_size     = 5
      desired_size = 2

      use_mixed_instances_policy = true
      mixed_instances_policy = {
        instances_distribution = {
          on_demand_base_capacity                  = 0
          on_demand_percentage_above_base_capacity = 10
          spot_allocation_strategy                 = "capacity-optimized"
        }

        override = [
          {
            instance_type     = "m5.large"
            weighted_capacity = "1"
          },
          {
            instance_type     = "m6i.large"
            weighted_capacity = "2"
          },
        ]
      }
    }
  }

  # EKS Managed Node Group(s)
  eks_managed_node_group_defaults = {
    instance_types = ["m6i.large", "m5.large", "m5n.large", "m5zn.large"]
  }

  eks_managed_node_groups = {
    blue = {}
    green = {
      min_size     = 1
      max_size     = 10
      desired_size = 1

      instance_types = ["t3.large"]
      capacity_type  = "SPOT"
    }
  }

  # Fargate Profile(s)
  fargate_profiles = {
    default = {
      name = "default"
      selectors = [
        {
          namespace = "default"
        }
      ]
    }
  }

  # aws-auth configmap
  manage_aws_auth_configmap = true

  aws_auth_roles = [
    {
      rolearn  = "arn:aws:iam::66666666666:role/role1"
      username = "role1"
      groups   = ["system:masters"]
    },
  ]

  aws_auth_users = [
    {
      userarn  = "arn:aws:iam::66666666666:user/user1"
      username = "user1"
      groups   = ["system:masters"]
    },
    {
      userarn  = "arn:aws:iam::66666666666:user/user2"
      username = "user2"
      groups   = ["system:masters"]
    },
  ]

  aws_auth_accounts = [
    "777777777777",
    "888888888888",
  ]

  tags = {
    Environment = "dev"
    Terraform   = "true"
  }
}
```
### CloudWatch:
Terraform AWS CloudWatch module by Terraform AWS modules [CloudWatch](https://registry.terraform.io/modules/terraform-aws-modules/cloudwatch/aws/latest)

### ElasticCache:
Terraform AWS ElasticCache module by Terraform AWS modules [ElasticCache Redis](https://registry.terraform.io/modules/cloudposse/elasticache-redis/aws/latest)
### IAM:
Terraform AWS IAM module by Terraform AWS modules [IAM](https://registry.terraform.io/modules/terraform-aws-modules/iam/aws/latest)
### Vault
If we use Vault HA where vault uses consul as storage backend, then we can provision all vault data through consul.
