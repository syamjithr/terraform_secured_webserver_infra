# terraform_secured_webserver_infra
## A simple and secured infrastructure using terraform in AWS. It Creates a VPC with three public and private subnets. Deploys three ec2 instances, One Bastion, One web server, and one database server. The bastion and webserver are created in the public subnet and the database server in the private subnet. The web server and database allow only the bastion server to ssh into it.
### Reference:
https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/vpc
### Prerequisite:
Download Terraform from link and set up it in a linux machine. ( Here I'm using a ec2 instance) Configure aws cli in the instance.
### First we need to create a directory 
##### mkdir ec2-vpc
##### cd ec2-vpc/
##### terraform init
##### vim ec2-web-infra.tf
```
#########################################################
# vpc creation
#########################################################

resource "aws_vpc" "blog" {

  cidr_block       = var.vpc.cidr
  enable_dns_support = true
  enable_dns_hostnames = true
  tags = {
    Name = var.vpc.name
  }
}


#########################################################
# igw creation
#########################################################

resource "aws_internet_gateway" "blog" {

  vpc_id = aws_vpc.blog.id
  tags = {
    Name = var.vpc.name
  }
}

#########################################################
# public subnet1
#########################################################

resource "aws_subnet" "public1" {

  vpc_id                    = aws_vpc.blog.id
  cidr_block                = var.public1.cidr
  availability_zone         = var.public1.az
  map_public_ip_on_launch   = true
  tags = {
    Name = "${var.vpc.name}-public1"
  }
}

#########################################################
# public subnet2
#########################################################

resource "aws_subnet" "public2" {

  vpc_id                    = aws_vpc.blog.id
  cidr_block                = var.public2.cidr
  availability_zone         = var.public2.az
  map_public_ip_on_launch   = true
  tags = {
    Name = "${var.vpc.name}-public2"
  }
}

#########################################################
# public subnet3
#########################################################

resource "aws_subnet" "public3" {

  vpc_id                    = aws_vpc.blog.id
  cidr_block                = var.public3.cidr
  availability_zone         = var.public3.az
  map_public_ip_on_launch   = true
  tags = {
    Name = "${var.vpc.name}-public3"
  }
}

#########################################################
# private subnet1
#########################################################

resource "aws_subnet" "private1" {

  vpc_id                    = aws_vpc.blog.id
  cidr_block                = var.private1.cidr
  availability_zone         = var.private1.az
  map_public_ip_on_launch   = false
  tags = {
    Name = "${var.vpc.name}-private1"
  }
}

#########################################################
# private subnet2
#########################################################

resource "aws_subnet" "private2" {

  vpc_id                    = aws_vpc.blog.id
  cidr_block                = var.private2.cidr
  availability_zone         = var.private2.az
  map_public_ip_on_launch   = false
  tags = {
    Name = "${var.vpc.name}-private2"
  }
}

#########################################################
# private subnet3
#########################################################

resource "aws_subnet" "private3" {

  vpc_id                    = aws_vpc.blog.id
  cidr_block                = var.private3.cidr
  availability_zone         = var.private3.az
  map_public_ip_on_launch   = false
  tags = {
    Name = "${var.vpc.name}-private3"
  }
}


#########################################################
# public route table
#########################################################

resource "aws_route_table" "public" {

  vpc_id        = aws_vpc.blog.id
  route {
    cidr_block  = "0.0.0.0/0"
    gateway_id  = aws_internet_gateway.blog.id
  }

  tags = {
    Name = "${var.vpc.name}-public"
  }
}

#########################################################
# public1 subnet to public route
#########################################################

resource "aws_route_table_association" "public1" {
  subnet_id      = aws_subnet.public1.id
  route_table_id = aws_route_table.public.id
}

#########################################################
# public2 subnet to public route
#########################################################

resource "aws_route_table_association" "public2" {
  subnet_id      = aws_subnet.public2.id
  route_table_id = aws_route_table.public.id
}

#########################################################
# public3 subnet to public route
#########################################################

resource "aws_route_table_association" "public3" {
  subnet_id      = aws_subnet.public3.id
  route_table_id = aws_route_table.public.id
}


#########################################################
# Elastic Ip
#########################################################
resource "aws_eip" "nat" {
  vpc      = true
}


#########################################################
# Nat gateway
#########################################################

resource "aws_nat_gateway" "blog" {
  allocation_id = aws_eip.nat.id
  subnet_id     = aws_subnet.public1.id
  tags = {
    Name = "${var.vpc.name}-vpc"
  }
}


#########################################################
# private route table
#########################################################

resource "aws_route_table" "private" {

  vpc_id            = aws_vpc.blog.id
  route {
    cidr_block      = "0.0.0.0/0"
    nat_gateway_id  = aws_nat_gateway.blog.id
  }

  tags = {
    Name = "${var.vpc.name}-private"
  }
}

#########################################################
# private1 subnet to private route
#########################################################

resource "aws_route_table_association" "private1" {
  subnet_id      = aws_subnet.private1.id
  route_table_id = aws_route_table.private.id
}

#########################################################
# private2 subnet to private route
#########################################################

resource "aws_route_table_association" "private2" {
  subnet_id      = aws_subnet.private2.id
  route_table_id = aws_route_table.private.id
}

#########################################################
# private3 sbent to private route
#########################################################

resource "aws_route_table_association" "private3" {
  subnet_id      = aws_subnet.private3.id
  route_table_id = aws_route_table.private.id
}


##########################################################
# Bastion Security Group
##########################################################

resource "aws_security_group" "bastion" {

  name        = "${var.vpc.name}-bastion"
  description = "allows 22 traffic"
  vpc_id      = aws_vpc.blog.id

  ingress {
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = {
    Name = "${var.vpc.name}-bastion"
  }
}

##########################################################
# Webserver Security Group
##########################################################

resource "aws_security_group" "webserver" {

  name        = "${var.vpc.name}-webserver"
  description = "allows 22,80,443 traffic"
  vpc_id      = aws_vpc.blog.id

  ingress {
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    security_groups = [ aws_security_group.bastion.id ]
  }
  
  ingress {
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    from_port   = 443
    to_port     = 443
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = {
    Name = "${var.vpc.name}-webserver"
  }
}


##########################################################
# Database Security Group
##########################################################

resource "aws_security_group" "database" {

  name        = "${var.vpc.name}-database"
  description = "allows 22,3306 traffic"
  vpc_id      = aws_vpc.blog.id

  ingress {
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    security_groups = [ aws_security_group.bastion.id ]
  }
  
  ingress {
    from_port   = 3306
    to_port     = 3306
    protocol    = "tcp"
    security_groups = [ aws_security_group.webserver.id ]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = {
    Name = "${var.vpc.name}-database"
  }
}


##########################################################
# keypair
##########################################################

resource "aws_key_pair" "keypair" {
  key_name   = "${var.vpc.name}-key"
  public_key = "-------------replace with public key---------------------"
  tags = {
    Name = "${var.vpc.name}-key"
  }
}

##########################################################
# bastion ec2
##########################################################

resource "aws_instance" "bastion" {
    
  ami                         = var.ami
  instance_type               = var.type
  key_name                    = aws_key_pair.keypair.key_name
  vpc_security_group_ids      = [ aws_security_group.bastion.id ]
  subnet_id                   = aws_subnet.public1.id
  tags = {
    Name = "${var.vpc.name}-bastion"
  }
}

##########################################################
# webserver ec2
##########################################################

resource "aws_instance" "webserver" {
    
  ami                         = var.ami
  instance_type               = var.type
  key_name                    = aws_key_pair.keypair.key_name
  vpc_security_group_ids      = [ aws_security_group.webserver.id ]
  subnet_id                   = aws_subnet.public2.id
  tags = {
    Name = "${var.vpc.name}-webserver"
  }
}

##########################################################
# database ec2
##########################################################

resource "aws_instance" "database" {
    
  ami                         = var.ami
  instance_type               = var.type
  key_name                    = aws_key_pair.keypair.key_name
  vpc_security_group_ids      = [ aws_security_group.database.id ]
  subnet_id                   = aws_subnet.private1.id
  tags = {
    Name = "${var.vpc.name}-database"
  }
}
```
##### variables files
vim ec2-var.tf
```
variable "ami" {
  default = "ami-09246ddb00c7c4fef"	
}

variable "type" {
  default = "t2.micro"
}
```
vim vpc-var.tf
```
variable "vpc" {
  type = map
  default = {
    "cidr"  = "172.16.0.0/16"
    "name"  = "blog"
  }
}

variable "public1" {
  type = map
  default = {
    "cidr" = "172.16.0.0/19" 
    "az" = "us-east-2a"
  }
}

variable "public2" {
  type = map
  default = {
    "cidr" = "172.16.32.0/19" 
    "az" = "us-east-2b"
  }
}

variable "public3" {
  type = map
  default = {
    "cidr" = "172.16.64.0/19" 
    "az" = "us-east-2c"
  }
}


variable "private1" {
  type = map
  default = {
    "cidr" = "172.16.96.0/19" 
    "az" = "us-east-2a"
  }
}

variable "private2" {
  type = map
  default = {
    "cidr" = "172.16.128.0/19" 
    "az" = "us-east-2b"
  }
}

variable "private3" {
  type = map
  default = {
    "cidr" = "172.16.160.0/19" 
    "az" = "us-east-2c"
  }
}
```
##### Execution
```
#terraform validate   - syntax check 

#terraform plan - Creating an execution plan ( to check what will get installed before running it)

# terraform apply - Applying

# terraform destroy - Destroying what we have applied through terrafrom apply

We can also use -auto-approve while applying.

# terraform apply -auto-approve
```
