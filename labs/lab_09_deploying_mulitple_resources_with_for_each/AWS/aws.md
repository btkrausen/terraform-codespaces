# LAB-09-AWS: Creating and Managing Resources with the For_Each Meta-Argument

## Overview
In this lab, you will learn how to use Terraform's `for_each` meta-argument to create and manage multiple resources efficiently. You'll discover how `for_each` differs from `count` and when to use each approach. The lab uses AWS free-tier resources to ensure no costs are incurred.

[![Lab 09](https://github.com/btkrausen/terraform-testing/actions/workflows/aws_lab_validation.yml/badge.svg?branch=main)](https://github.com/btkrausen/terraform-testing/actions/workflows/aws_lab_validation.yml)

**Preview Mode**: Use `Cmd/Ctrl + Shift + V` in VSCode to see a nicely formatted version of this lab!

## Prerequisites
- Terraform installed
- AWS free tier account
- Basic understanding of Terraform and AWS concepts
- Familiarity with the `count` meta-argument

Note: AWS credentials are required for this lab.

## How to Use This Hands-On Lab

1. **Create a Codespace** from this repo (click the button below).  
2. Once the Codespace is running, open the integrated terminal.
3. Follow the instructions in each **lab** to complete the exercises.

[![Open in GitHub Codespaces](https://github.com/codespaces/badge.svg)](https://codespaces.new/btkrausen/terraform-codespaces)

## Estimated Time
45 minutes

## Existing Configuration Files

The lab directory contains the following files with resources created using `count` that we'll refactor to use `for_each`:

 - `main.tf`
 - `variables.tf`
 - `providers.tf`

Examine these files and notice:
- Subnet creation using count and list indexing
- Security group resources using count
- The potential issues if list elements are reordered or removed

## Lab Steps

### 1. Configure AWS Credentials

Set up your AWS credentials as environment variables:

```bash
export AWS_ACCESS_KEY_ID="your_access_key"
export AWS_SECRET_ACCESS_KEY="your_secret_key"
```

### 2. Update Variables for For_Each

Modify `variables.tf` to include a map and set variables for use with `for_each`:

```hcl
# New map variables for for_each
variable "subnet_config" {
  description = "Map of subnet configurations"
  type        = map(string)
  default = {
    "public"   = "10.0.10.0/24"
    "private1" = "10.0.20.0/24"
    "private2" = "10.0.30.0/24"
  }
}

variable "subnet_azs" {
  description = "Map of subnet availability zones"
  type        = map(string)
  default = {
    "public"   = "us-east-1a"
    "private1" = "us-east-1b"
    "private2" = "us-east-1c"
  }
}

variable "security_group_config" {
  description = "Map of security group ports"
  type        = map(number)
  default = {
    "web" = 80
    "app" = 8080
    "db"  = 3306
  }
}
```

### 3. Keep Count-Based Resources

In your `main.tf`, leave the following VPC and count-based resources in place for comparison:

```hcl
# Main VPC
resource "aws_vpc" "main" {
  cidr_block           = "10.0.0.0/16"
  enable_dns_hostnames = true
  enable_dns_support   = true

  tags = {
    Name = "main-vpc"
  }
}

# Subnets created with count (for comparison)
resource "aws_subnet" "subnet_count" {
  count             = 3
  vpc_id            = aws_vpc.main.id
  cidr_block        = var.subnet_config[sort(keys(var.subnet_config))[count.index]
  availability_zone = var.subnet_azs[sort(keys(var.subnet_config))[count.index]]

  tags = {
    Name = "subnet-count-${count.index + 1}"
    Tier = count.index < 1 ? "public" : "private"
  }
}
```

### 4. Add Subnet Resources Using For_Each

Add new subnet resources using for_each to `main.tf`:

```hcl
# Subnets created with for_each
resource "aws_subnet" "subnet_foreach" {
  for_each          = var.subnet_config
  vpc_id            = aws_vpc.main.id
  cidr_block        = each.value
  availability_zone = var.subnet_azs[each.key]

  tags = {
    Name = "subnet-${each.key}"
    Tier = "standard"
  }
}
```

### 5. Apply and Compare Subnets

Initialize and apply the configuration:

```bash
terraform init
terraform plan
terraform apply
```

Compare the `count`-based and `for_each`-based subnets in the AWS Console:
- Notice how the `for_each` subnets have meaningful names based on map keys
- Observe how the resources are referenced differently in the state file

### 6. Add Security Group Resources Using For_Each

Add new security group resources using for_each:

```hcl
# Security groups created with for_each
resource "aws_security_group" "sg_foreach" {
  for_each    = var.security_group_config
  name        = "${each.key}-sg-foreach"
  description = "Security group for ${each.key} servers"
  vpc_id      = aws_vpc.main.id

  tags = {
    Name = "${each.key}-sg-foreach"
  }
}

# Security group rules created with for_each
resource "aws_security_group_rule" "ingress_foreach" {
  for_each          = var.security_group_config
  type              = "ingress"
  from_port         = each.value
  to_port           = each.value
  protocol          = "tcp"
  cidr_blocks       = ["0.0.0.0/0"]
  security_group_id = aws_security_group.sg_foreach[each.key].id
}
```

Apply the configuration:

```bash
terraform plan
terraform apply
```

### 7. Demonstrate For_Each with a Simple Map

Add a new variable for demonstration with a simple map:

```hcl
variable "route_tables" {
  description = "Map of route tables to create"
  type        = map(string)
  default     = {
    "public"   = "Public route table"
    "private1" = "Private route table 1"
    "private2" = "Private route table 2"
  }
}
```

Add route tables using `for_each` with this map:

```hcl
# Route tables created with for_each and a simple map
resource "aws_route_table" "rt" {
  for_each = var.route_tables
  vpc_id   = aws_vpc.main.id

  tags = {
    Name        = "${each.key}-rt"
    Description = each.value
  }
}
```

Apply the configuration:

```bash
terraform plan
terraform apply
```

### 8. Create Outputs Using For_Each

Create an `outputs.tf` file to demonstrate how to reference `for_each`-based resources:

```hcl
output "vpc_id" {
  description = "The ID of the VPC"
  value       = aws_vpc.main.id
}

# Outputs for count-based resources
output "subnet_count_ids" {
  description = "The IDs of the count-based subnets"
  value       = aws_subnet.subnet[*].id
}

# Outputs for for_each-based resources (map)
output "subnet_foreach_ids" {
  description = "The IDs of the for_each-based subnets"
  value       = aws_subnet.subnet_foreach
}

output "security_group_foreach_ids" {
  description = "The IDs of the for_each-based security groups"
  value       = aws_security_group.sg_foreach
}

# Outputs for simple map-based resources
output "route_table_ids" {
  description = "The IDs of the map-based route tables"
  value       = aws_route_table.rt
}
```

Apply the configuration to see the outputs:

```bash
terraform apply -auto-approve
```

### 9. Experiment by Modifying Resources

Let's demonstrate the advantage of for_each when removing or renaming resources:

1. Modify the `subnet_config` variable to remove one subnet:

```hcl
variable "subnet_config" {
  description = "Map of subnet configurations"
  type        = map(string)
  default = {
    "public"   = "10.0.1.0/24"
    "private1" = "10.0.2.0/24"
                              # <-- Removed "private2" subnet
  }
}

# Also update subnet_azs
variable "subnet_azs" {
  description = "Map of subnet availability zones"
  type        = map(string)
  default = {
    "public"   = "us-east-1a"
    "private1" = "us-east-1b"
                              # <-- Removed "private2" subnet AZ
  }
}
```

2. Also modify the `subnet_cidr_blocks` variable to remove an element:

```hcl
variable "subnet_cidr_blocks" {
  description = "CIDR blocks for subnets"
  type        = list(string)
  default     = ["10.0.1.0/24", "10.0.2.0/24"] # <-- Removed the third element
}
```

Finally, update the `aws_subnet.subnet` resource and change the `count` to `2` since we removed one of the elements from the variable:

```hcl
# Subnets created with count
resource "aws_subnet" "subnet" {
  count             = 2                  # <-- Update the count from 3 to 2
  vpc_id            = aws_vpc.main.id
  cidr_block        = var.subnet_cidr_blocks[count.index]
  availability_zone = var.availability_zones[count.index]

  tags = {
    Name = "subnet-${count.index + 1}"
    Tier = count.index < 1 ? "public" : "private"
  }
}
```

Apply the changes and observe the differences:

```bash
terraform plan
```

Notice how:
- With count, removing the third element shifts all indexes, potentially recreating resources
- With for_each, only the specific "private2" subnet is removed, leaving others untouched

### 10. Add a New Resource to Existing Map

Add a new entry to the `security_group_config` map:

```hcl
variable "security_group_config" {
  description = "Map of security group ports"
  type        = map(number)
  default = {
    "web"   = 80
    "app"   = 8080
    "db"    = 3306
    "cache" = 6379  # Added new entry
  }
}
```

Apply the changes:

```bash
terraform plan
terraform apply
```

Notice how only the new resource is added without affecting existing ones.

### 11. Clean Up

Remove all created resources:

```bash
terraform destroy
```

## Understanding For_Each

Let's examine how for_each improves your Terraform configurations:

### For_Each vs Count
- **For_Each**: Resources are indexed by key (string) instead of numeric index
- **Count**: Resources are indexed by position (0, 1, 2, ...)

### For_Each Advantages
- Resources maintain stable identity when items are added or removed
- Keys provide meaningful naming in state and console
- More expressive and clear configuration
- Better handles non-uniform resource configurations

### For_Each Usage
- Can use a map with string keys
- With a basic map of strings: `for_each = var.route_tables`
- With a map of numbers: `for_each = var.security_group_config`

### Resource References
- Referencing a specific resource: `aws_subnet.subnet_foreach["public"]`
- Referencing a value from a specific resource: `aws_subnet.subnet_foreach["public"].id`
- Iterating over all resources: `for k, v in aws_subnet.subnet_foreach : k => v.id`

## Additional Exercises

1. Create multiple IAM users with for_each
2. Add subnet-route table associations using for_each
3. Create multiple S3 buckets with different configurations
4. Try converting other count-based resources to for_each

## Common Issues and Solutions

1. **Invalid for_each Value**
   - For_each value must be a map or set of strings
   - Lists must be converted with `toset()`
   - Map values must be known at plan time

2. **Key Type Errors**
   - For_each keys must be strings
   - Numeric keys in maps should be quoted

3. **Resource References**
   - Access for_each resources with square brackets and the key
   - Don't use asterisk notation (like with count)
