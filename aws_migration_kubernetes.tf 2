provider "aws" {
  region = "us-west-2"
}

resource "aws_vpc" "migration_vpc" {
  cidr_block = "10.1.0.0/16"
}

resource "aws_subnet" "migration_subnet" {
  vpc_id     = aws_vpc.migration_vpc.id
  cidr_block = "10.1.1.0/24"
}

resource "aws_route_table" "migration_route_table" {
  vpc_id = aws_vpc.migration_vpc.id
}

resource "aws_route_table_association" "migration_route_assoc" {
  subnet_id      = aws_subnet.migration_subnet.id
  route_table_id = aws_route_table.migration_route_table.id
}

resource "aws_instance" "migrated_server" {
  ami           = "ami-0abcdef1234567890"
  instance_type = "t3.medium"
  subnet_id     = aws_subnet.migration_subnet.id
  key_name      = "migration-key"
}

resource "aws_s3_bucket" "migration_bucket" {
  bucket = "migration-data-bucket"
  acl    = "private"
}

resource "aws_eks_cluster" "migration_eks" {
  name     = "migration-eks-cluster"
  role_arn = aws_iam_role.eks_role.arn

  vpc_config {
    subnet_ids = [aws_subnet.migration_subnet.id]
  }
}

resource "aws_iam_role" "eks_role" {
  name = "migration-eks-role"
  
  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [{
      Action = "sts:AssumeRole"
      Effect = "Allow"
      Principal = {
        Service = "eks.amazonaws.com"
      }
    }]
  })
}

resource "aws_iam_role_policy_attachment" "eks_policy" {
  role       = aws_iam_role.eks_role.name
  policy_arn = "arn:aws:iam::aws:policy/AmazonEKSClusterPolicy"
}

variable "db_password" {
  description = "Database password for RDS"
  type        = string
  sensitive   = true
}