---
title : "Cài đặt Terraform và kiến trúc mạng VPC"
date :  "`r Sys.Date()`" 
weight : 4 
chapter : false
pre : " <b> 2.4 </b> "
---

1. Cài đặt Terraform
```
sudo yum install -y yum-utils
sudo yum-config-manager --add-repo https://rpm.releases.hashicorp.com/AmazonLinux/hashicorp.repo
sudo yum install terraform -y

```
![CREATENETWORKING](/images/2.prerequisite/001-installterraform.png)

2. Cài đặt module terraform networking từ github

```
git clone https://github.com/leonli/terraform-vpc-networking 
cd terraform-vpc-networking
```
Dùng lệnh dưới đây truy cập file **main.tf** đổi ***vpc = true*** thành ***domain = "vpc"***
```
sudo vim modules/networking/main.tf
```
![CREATENETWORKING](/images/2.prerequisite/001-installterraform.png)
3. Tạo các tham số cần thiết cho terraform.tfvars để xây dựng kiến ​​trúc mạng VPC.

```
cat > terraform.tfvars <<EOF
//AWS 
region      = "ap-southeast-1"
environment = "k8s"

/* module networking */
vpc_cidr             = "10.0.0.0/16"
public_subnets_cidr  = ["10.0.1.0/24", "10.0.3.0/24", "10.0.5.0/24"] //List of Public subnet cidr range
private_subnets_cidr = ["10.0.2.0/24", "10.0.4.0/24", "10.0.6.0/24"] //List of private subnet cidr range
EOF
```

4. Khởi tạo địa hình và áp dụng mẫu để tạo VPC và các tài nguyên liên quan.

```
terraform init
```
![CREATENETWORKING](/images/2.prerequisite/002-installterraform.png)
```
terraform plan
```
```
ec2-user:~/environment/terraform-vpc-networking (master) $ terraform plan

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # module.networking.aws_eip.nat_eip will be created
  + resource "aws_eip" "nat_eip" {
      + allocation_id        = (known after apply)
      + association_id       = (known after apply)
      + carrier_ip           = (known after apply)
      + customer_owned_ip    = (known after apply)
      + domain               = (known after apply)
      + id                   = (known after apply)
      + instance             = (known after apply)
      + network_border_group = (known after apply)
      + network_interface    = (known after apply)
      + private_dns          = (known after apply)
      + private_ip           = (known after apply)
      + public_dns           = (known after apply)
      + public_ip            = (known after apply)
      + public_ipv4_pool     = (known after apply)
      + tags_all             = (known after apply)
      + vpc                  = true
    }

  # module.networking.aws_internet_gateway.ig will be created
  + resource "aws_internet_gateway" "ig" {
      + arn      = (known after apply)
      + id       = (known after apply)
      + owner_id = (known after apply)
      + tags     = {
          + "Environment" = "k8s"
          + "Name"        = "k8s-igw"
        }
      + tags_all = {
          + "Environment" = "k8s"
          + "Name"        = "k8s-igw"
        }
      + vpc_id   = (known after apply)
    }

  # module.networking.aws_nat_gateway.nat will be created
  + resource "aws_nat_gateway" "nat" {
      + allocation_id        = (known after apply)
      + association_id       = (known after apply)
      + connectivity_type    = "public"
      + id                   = (known after apply)
      + network_interface_id = (known after apply)
      + private_ip           = (known after apply)
      + public_ip            = (known after apply)
      + subnet_id            = (known after apply)
      + tags                 = {
          + "Environment" = "k8s"
          + "Name"        = "nat"
        }
      + tags_all             = {
          + "Environment" = "k8s"
          + "Name"        = "nat"
        }
    }

  # module.networking.aws_route.private_nat_gateway will be created
  + resource "aws_route" "private_nat_gateway" {
      + destination_cidr_block = "0.0.0.0/0"
      + id                     = (known after apply)
      + instance_id            = (known after apply)
      + instance_owner_id      = (known after apply)
      + nat_gateway_id         = (known after apply)
      + network_interface_id   = (known after apply)
      + origin                 = (known after apply)
      + route_table_id         = (known after apply)
      + state                  = (known after apply)
    }

  # module.networking.aws_route.public_internet_gateway will be created
  + resource "aws_route" "public_internet_gateway" {
      + destination_cidr_block = "0.0.0.0/0"
      + gateway_id             = (known after apply)
      + id                     = (known after apply)
      + instance_id            = (known after apply)
      + instance_owner_id      = (known after apply)
      + network_interface_id   = (known after apply)
      + origin                 = (known after apply)
      + route_table_id         = (known after apply)
      + state                  = (known after apply)
    }

  # module.networking.aws_route_table.private will be created
  + resource "aws_route_table" "private" {
      + arn              = (known after apply)
      + id               = (known after apply)
      + owner_id         = (known after apply)
      + propagating_vgws = (known after apply)
      + route            = (known after apply)
      + tags             = {
          + "Environment" = "k8s"
          + "Name"        = "k8s-private-route-table"
        }
      + tags_all         = {
          + "Environment" = "k8s"
          + "Name"        = "k8s-private-route-table"
        }
      + vpc_id           = (known after apply)
    }

  # module.networking.aws_route_table.public will be created
  + resource "aws_route_table" "public" {
      + arn              = (known after apply)
      + id               = (known after apply)
      + owner_id         = (known after apply)
      + propagating_vgws = (known after apply)
      + route            = (known after apply)
      + tags             = {
          + "Environment" = "k8s"
          + "Name"        = "k8s-public-route-table"
        }
      + tags_all         = {
          + "Environment" = "k8s"
          + "Name"        = "k8s-public-route-table"
        }
      + vpc_id           = (known after apply)
    }

  # module.networking.aws_route_table_association.private[0] will be created
  + resource "aws_route_table_association" "private" {
      + id             = (known after apply)
      + route_table_id = (known after apply)
      + subnet_id      = (known after apply)
    }

  # module.networking.aws_route_table_association.private[1] will be created
  + resource "aws_route_table_association" "private" {
      + id             = (known after apply)
      + route_table_id = (known after apply)
      + subnet_id      = (known after apply)
    }

  # module.networking.aws_route_table_association.private[2] will be created
  + resource "aws_route_table_association" "private" {
      + id             = (known after apply)
      + route_table_id = (known after apply)
      + subnet_id      = (known after apply)
    }

  # module.networking.aws_route_table_association.public[0] will be created
  + resource "aws_route_table_association" "public" {
      + id             = (known after apply)
      + route_table_id = (known after apply)
      + subnet_id      = (known after apply)
    }

  # module.networking.aws_route_table_association.public[1] will be created
  + resource "aws_route_table_association" "public" {
      + id             = (known after apply)
      + route_table_id = (known after apply)
      + subnet_id      = (known after apply)
    }

  # module.networking.aws_route_table_association.public[2] will be created
  + resource "aws_route_table_association" "public" {
      + id             = (known after apply)
      + route_table_id = (known after apply)
      + subnet_id      = (known after apply)
    }

  # module.networking.aws_security_group.default will be created
  + resource "aws_security_group" "default" {
      + arn                    = (known after apply)
      + description            = "Default security group to allow inbound/outbound from the VPC"
      + egress                 = [
          + {
              + cidr_blocks      = []
              + description      = ""
              + from_port        = 0
              + ipv6_cidr_blocks = []
              + prefix_list_ids  = []
              + protocol         = "-1"
              + security_groups  = []
              + self             = true
              + to_port          = 0
            },
        ]
      + id                     = (known after apply)
      + ingress                = [
          + {
              + cidr_blocks      = []
              + description      = ""
              + from_port        = 0
              + ipv6_cidr_blocks = []
              + prefix_list_ids  = []
              + protocol         = "-1"
              + security_groups  = []
              + self             = true
              + to_port          = 0
            },
        ]
      + name                   = "k8s-default-sg"
      + name_prefix            = (known after apply)
      + owner_id               = (known after apply)
      + revoke_rules_on_delete = false
      + tags                   = {
          + "Environment" = "k8s"
        }
      + tags_all               = {
          + "Environment" = "k8s"
        }
      + vpc_id                 = (known after apply)
    }

  # module.networking.aws_subnet.private_subnet[0] will be created
  + resource "aws_subnet" "private_subnet" {
      + arn                                            = (known after apply)
      + assign_ipv6_address_on_creation                = false
      + availability_zone                              = "ap-southeast-1a"
      + availability_zone_id                           = (known after apply)
      + cidr_block                                     = "10.0.2.0/24"
      + enable_dns64                                   = false
      + enable_resource_name_dns_a_record_on_launch    = false
      + enable_resource_name_dns_aaaa_record_on_launch = false
      + id                                             = (known after apply)
      + ipv6_cidr_block_association_id                 = (known after apply)
      + ipv6_native                                    = false
      + map_public_ip_on_launch                        = false
      + owner_id                                       = (known after apply)
      + private_dns_hostname_type_on_launch            = (known after apply)
      + tags                                           = {
          + "Environment" = "k8s"
          + "Name"        = "k8s-ap-southeast-1a-private-subnet"
        }
      + tags_all                                       = {
          + "Environment" = "k8s"
          + "Name"        = "k8s-ap-southeast-1a-private-subnet"
        }
      + vpc_id                                         = (known after apply)
    }

  # module.networking.aws_subnet.private_subnet[1] will be created
  + resource "aws_subnet" "private_subnet" {
      + arn                                            = (known after apply)
      + assign_ipv6_address_on_creation                = false
      + availability_zone                              = "ap-southeast-1b"
      + availability_zone_id                           = (known after apply)
      + cidr_block                                     = "10.0.4.0/24"
      + enable_dns64                                   = false
      + enable_resource_name_dns_a_record_on_launch    = false
      + enable_resource_name_dns_aaaa_record_on_launch = false
      + id                                             = (known after apply)
      + ipv6_cidr_block_association_id                 = (known after apply)
      + ipv6_native                                    = false
      + map_public_ip_on_launch                        = false
      + owner_id                                       = (known after apply)
      + private_dns_hostname_type_on_launch            = (known after apply)
      + tags                                           = {
          + "Environment" = "k8s"
          + "Name"        = "k8s-ap-southeast-1b-private-subnet"
        }
      + tags_all                                       = {
          + "Environment" = "k8s"
          + "Name"        = "k8s-ap-southeast-1b-private-subnet"
        }
      + vpc_id                                         = (known after apply)
    }

  # module.networking.aws_subnet.private_subnet[2] will be created
  + resource "aws_subnet" "private_subnet" {
      + arn                                            = (known after apply)
      + assign_ipv6_address_on_creation                = false
      + availability_zone                              = "ap-southeast-1c"
      + availability_zone_id                           = (known after apply)
      + cidr_block                                     = "10.0.6.0/24"
      + enable_dns64                                   = false
      + enable_resource_name_dns_a_record_on_launch    = false
      + enable_resource_name_dns_aaaa_record_on_launch = false
      + id                                             = (known after apply)
      + ipv6_cidr_block_association_id                 = (known after apply)
      + ipv6_native                                    = false
      + map_public_ip_on_launch                        = false
      + owner_id                                       = (known after apply)
      + private_dns_hostname_type_on_launch            = (known after apply)
      + tags                                           = {
          + "Environment" = "k8s"
          + "Name"        = "k8s-ap-southeast-1c-private-subnet"
        }
      + tags_all                                       = {
          + "Environment" = "k8s"
          + "Name"        = "k8s-ap-southeast-1c-private-subnet"
        }
      + vpc_id                                         = (known after apply)
    }

  # module.networking.aws_subnet.public_subnet[0] will be created
  + resource "aws_subnet" "public_subnet" {
      + arn                                            = (known after apply)
      + assign_ipv6_address_on_creation                = false
      + availability_zone                              = "ap-southeast-1a"
      + availability_zone_id                           = (known after apply)
      + cidr_block                                     = "10.0.1.0/24"
      + enable_dns64                                   = false
      + enable_resource_name_dns_a_record_on_launch    = false
      + enable_resource_name_dns_aaaa_record_on_launch = false
      + id                                             = (known after apply)
      + ipv6_cidr_block_association_id                 = (known after apply)
      + ipv6_native                                    = false
      + map_public_ip_on_launch                        = true
      + owner_id                                       = (known after apply)
      + private_dns_hostname_type_on_launch            = (known after apply)
      + tags                                           = {
          + "Environment" = "k8s"
          + "Name"        = "k8s-ap-southeast-1a-public-subnet"
        }
      + tags_all                                       = {
          + "Environment" = "k8s"
          + "Name"        = "k8s-ap-southeast-1a-public-subnet"
        }
      + vpc_id                                         = (known after apply)
    }

  # module.networking.aws_subnet.public_subnet[1] will be created
  + resource "aws_subnet" "public_subnet" {
      + arn                                            = (known after apply)
      + assign_ipv6_address_on_creation                = false
      + availability_zone                              = "ap-southeast-1b"
      + availability_zone_id                           = (known after apply)
      + cidr_block                                     = "10.0.3.0/24"
      + enable_dns64                                   = false
      + enable_resource_name_dns_a_record_on_launch    = false
      + enable_resource_name_dns_aaaa_record_on_launch = false
      + id                                             = (known after apply)
      + ipv6_cidr_block_association_id                 = (known after apply)
      + ipv6_native                                    = false
      + map_public_ip_on_launch                        = true
      + owner_id                                       = (known after apply)
      + private_dns_hostname_type_on_launch            = (known after apply)
      + tags                                           = {
          + "Environment" = "k8s"
          + "Name"        = "k8s-ap-southeast-1b-public-subnet"
        }
      + tags_all                                       = {
          + "Environment" = "k8s"
          + "Name"        = "k8s-ap-southeast-1b-public-subnet"
        }
      + vpc_id                                         = (known after apply)
    }

  # module.networking.aws_subnet.public_subnet[2] will be created
  + resource "aws_subnet" "public_subnet" {
      + arn                                            = (known after apply)
      + assign_ipv6_address_on_creation                = false
      + availability_zone                              = "ap-southeast-1c"
      + availability_zone_id                           = (known after apply)
      + cidr_block                                     = "10.0.5.0/24"
      + enable_dns64                                   = false
      + enable_resource_name_dns_a_record_on_launch    = false
      + enable_resource_name_dns_aaaa_record_on_launch = false
      + id                                             = (known after apply)
      + ipv6_cidr_block_association_id                 = (known after apply)
      + ipv6_native                                    = false
      + map_public_ip_on_launch                        = true
      + owner_id                                       = (known after apply)
      + private_dns_hostname_type_on_launch            = (known after apply)
      + tags                                           = {
          + "Environment" = "k8s"
          + "Name"        = "k8s-ap-southeast-1c-public-subnet"
        }
      + tags_all                                       = {
          + "Environment" = "k8s"
          + "Name"        = "k8s-ap-southeast-1c-public-subnet"
        }
      + vpc_id                                         = (known after apply)
    }

  # module.networking.aws_vpc.vpc will be created
  + resource "aws_vpc" "vpc" {
      + arn                                  = (known after apply)
      + cidr_block                           = "10.0.0.0/16"
      + default_network_acl_id               = (known after apply)
      + default_route_table_id               = (known after apply)
      + default_security_group_id            = (known after apply)
      + dhcp_options_id                      = (known after apply)
      + enable_dns_hostnames                 = true
      + enable_dns_support                   = true
      + enable_network_address_usage_metrics = (known after apply)
      + id                                   = (known after apply)
      + instance_tenancy                     = "default"
      + ipv6_association_id                  = (known after apply)
      + ipv6_cidr_block                      = (known after apply)
      + ipv6_cidr_block_network_border_group = (known after apply)
      + main_route_table_id                  = (known after apply)
      + owner_id                             = (known after apply)
      + tags                                 = {
          + "Environment" = "k8s"
          + "Name"        = "k8s-vpc"
        }
      + tags_all                             = {
          + "Environment" = "k8s"
          + "Name"        = "k8s-vpc"
        }
    }

Plan: 21 to add, 0 to change, 0 to destroy.
```
```
terraform apply
```
```
ec2-user:~/environment/terraform-vpc-networking (master) $ terraform apply

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # module.networking.aws_eip.nat_eip will be created
  + resource "aws_eip" "nat_eip" {
      + allocation_id        = (known after apply)
      + association_id       = (known after apply)
      + carrier_ip           = (known after apply)
      + customer_owned_ip    = (known after apply)
      + domain               = (known after apply)
      + id                   = (known after apply)
      + instance             = (known after apply)
      + network_border_group = (known after apply)
      + network_interface    = (known after apply)
      + private_dns          = (known after apply)
      + private_ip           = (known after apply)
      + public_dns           = (known after apply)
      + public_ip            = (known after apply)
      + public_ipv4_pool     = (known after apply)
      + tags_all             = (known after apply)
      + vpc                  = true
    }

  # module.networking.aws_internet_gateway.ig will be created
  + resource "aws_internet_gateway" "ig" {
      + arn      = (known after apply)
      + id       = (known after apply)
      + owner_id = (known after apply)
      + tags     = {
          + "Environment" = "k8s"
          + "Name"        = "k8s-igw"
        }
      + tags_all = {
          + "Environment" = "k8s"
          + "Name"        = "k8s-igw"
        }
      + vpc_id   = (known after apply)
    }
...
...
...
Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes

module.networking.aws_vpc.vpc: Creating...
...
...
...
Apply complete! Resources: 21 added, 0 changed, 0 destroyed.

Outputs:

default_sg_id = "sg-04557a01e085df410"
private_subnets_id = [
  "subnet-0dee119ba55055966",
  "subnet-00ce94e6c6575771d",
  "subnet-05265621ecbce05ff",
]
public_route_table = "rtb-06d02f3e612dc7ec9"
public_subnets_id = [
  "subnet-0900df659ed1b977e",
  "subnet-02277901c9a13716f",
  "subnet-09540a9dc5d7c2052",
]
security_groups_ids = "sg-04557a01e085df410"
vpc_id = "vpc-07380ec3a9894314c"
```

5. Truy cập [giao diện quản trị dịch vụ VPC](https://ap-southeast-1.console.aws.amazon.com/vpc/home) để khám phá các cấu hình đã tạo
![CREATENETWORKING](/images/2.prerequisite/003-installterraform.png)