# terraform-aws-eks_addons-hpcc

# Overview 

This module creates HPCC AWS EKS cluster, with EFS File System storage.


***Important Note***
We integrated the work by [Ryan Rao, Wang Xiaoming, and Godji Fortil ](https://github.com/rrao4/HPCC-Platform/tree/HPCC-25955/helm/examples/efs) for storageclass and persistent volumes however, we do install our EFS CSI Driver as a part of EKS addon. (Read more on [EKS ADDON](https://registry.terraform.io/providers/figma/aws-4-49-0/latest/docs/resources/eks_addon))


**Important Note** Currently, this terraform modules and helm_release resource supports the static provisioning that has been worked on by other intern [Ryan Rao and his mentors, Wang Xiaoming, and Godji Fortil with HPCC version that their repo support](https://github.com/rrao4/HPCC-Platform/tree/HPCC-25955/helm/examples/efs). We have the static storage beyond Kubernetes. 

**Update** Now the work has been merged to [HPCC-candidate 2.9.x](https://github.com/hpcc-systems/HPCC-Platform/tree/candidate-9.2.x) you may want to do a github cloning and check out a particular branch to be able to use this application. 


### Providers
---

* AWS
* Kubernetes
* Helm

### Major Components / Modules for this project
---
1. AWS_VPC
2. AWS_EKS
3. AWS_EFS
4. Kubernetes

### Supported Arguments
---
Referencing [Terraform-hpcc-azure repository](https://github.com/hpcc-systems/terraform-azurerm-hpcc/tree/main), we have a set of input variables defined in order to instantiate our resources.

### ```Admin ``` Block 

| Name | Description    | Type    | Default | Required |
| :---:   | :---: | :---: |:---:|:---:|
|  name| name of the admin of the cluster   | string   | - | Yes|
|  email | email of the admin of the cluster   | string   | - | Yes|

The example usage looks as follows:

```
{
  "name": "example"
  "email": "example@lexisnexis.com"
}
```

This block describe the information of the user who is deploying the cluster. This block is used to tag resources. 

#### ```Networking``` Block  ```Object```

| Name | Description    | Type    | Default | Required |
| :---:   | :---: | :---: |:---:|:---:|
| cidr_block | cidr_blocks specification for vpc to be deployed | string | "10.0.0.0/16"| no
| region | aws region in which the resources will be deployed| string | "us-east-1" | no
| vpc_name | name of vpc | string | "custom_vpc" | no
| azs | availability zones | list(string) | ["us-east-1a", "us-east-1b"] | no
| public_subnets| subnet masks for the public subnet to be deployed. | list(string) | ["10.0.1.0/24", "10.0.2.0/24"] | no
| private_subnets | subnet masks for the private subnet to be deployed. | list(string) | ["10.0.3.0/24", "10.0.4.0/24"]| no

The example usage looks as follows:

```
networking = {
    cidr_block      = "10.0.0.0/16"
    region          = "us-east-1"
    vpc_name        = "custom-vpc"
    azs             = ["us-east-1a", "us-east-1b"]
    public_subnets  = ["10.0.1.0/24", "10.0.2.0/24"]
    private_subnets = ["10.0.3.0/24", "10.0.4.0/24"]
    nat_gateways    = true
  }
```


#### ```security_groups``` Block ```List of Objects```

VPC is associated with a set of security groups, and this module supports varied definiton of security groups. 
Arguments are:
| Name | Description    | Type    | Required |
| :---:   | :---: | :---: |:---:|
| name | name of security group | string | no
| description | description of the security group| string | no
| ingress | configuration block for ingress rule| list(object)| no 
| egress | configuration block for egress rule | list(object) | no

The example usage looks as follows:

```
security_groups = [{
    name        = "default-security-group"
    description = "Inbound & Outbound traffic for custom-security-group"
    ingress = [
      {
        description      = "Allow HTTPS"
        protocol         = "tcp"
        from_port        = 443
        to_port          = 443
        cidr_blocks      = ["0.0.0.0/0"]
        ipv6_cidr_blocks = null
      },
      {
        description      = "Allow HTTP"
        protocol         = "tcp"
        from_port        = 80
        to_port          = 80
        cidr_blocks      = ["0.0.0.0/0"]
        ipv6_cidr_blocks = null
      },
    ]
    egress = [
      {
        description      = "Allow all outbound traffic"
        protocol         = "-1"
        from_port        = 0
        to_port          = 0
        cidr_blocks      = ["0.0.0.0/0"]
        ipv6_cidr_blocks = ["::/0"]
      }
    ]
  }]
```

#### ```ec2-node``` block ```Object```

| Name | Description    | Type    | Default | 
| :---:   | :---: | :---: |:---:|
node_group_name | name of the node group | string | "default-node-group"
scaling_config_desired_size | number of desired nodes in the group | number | 6
scaling_config_max_size | maximum number of nodes in the group | number | 8
scaling_config_min_size | minimum number of nodes in the group | number | 4
ami_type | type of the amazon machine images to be used | string | "AL2_x86_64"
instance_types | type of instances to be created inside the node | list(string) | ["t3.medium"]
capacity_type | specifying that how we will be managing the resource and as a result how we will be chared. | string | "ON_DEMAND| 
disk_size | specifying root device disk size for the node group instance | number | 20


The example usage looks as follows:

```
ec2-node = {
    node_group_name             = "default-node-group"
    scaling_config_desired_size = 6
    scaling_config_max_size     = 8
    scaling_config_min_size     = 4
    ami_type                    = "AL2_x86_64"
    instance_types              = ["t3.medium"]
    capacity_type               = "ON_DEMAND"
    disk_size                   = 20
  }
```


#### ***Note on AWS provider configuration*** 
This module configures AWS with your locally stored aws profile under ```~/.aws/conf``` and ```~/.aws/credentials``` 
Variables associated with the configuration of aws are:
| Name | Description    | Type    | Default | Required |
| :---:   | :---: | :---: |:---:|:---:|
|profile| name of the IAM user profile to be used| string | - | yes | 
|path| path to the .aws directory | string | - | yes | 

#### Other variables

```cluster_name``` variable
| Name | Description    | Type    | Default | Required |
| :---:   | :---: | :---: |:---:|:---:|
| cluster_name | name of the cluster to be created| string | eks-cluster| no | 
| chart_path | path to the chart folder (locally or remotely) | string | - | yes | 



### Deploying a cluster

Make sure that you have kubectl installed so that you can check if the services is up and running correctly. 
1. First please make sure you have necessary providers cofigured by running:

```
terraform init
```

2. In order to deploy a cluster from this root module, make sure you have specified the required arguments and also optional arguments, if modified, in ```example/admin.tfvars``` file and the run the command:

```
terraform apply -var-file="example/admin.tfvars"
```

3. In order to destroy the infrastructure execute
```
terraform destroy -var-file="example/admin.tfvars"
```

