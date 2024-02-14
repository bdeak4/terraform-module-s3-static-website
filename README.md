# terraform-module-s3-static-website

Setup static website on AWS S3 bucket and CloudFront CDN

## How to use

```tf
terraform {
  required_version = ">= 1.0.0, < 2.0.0"

  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }

    cloudflare = {
      source  = "cloudflare/cloudflare"
      version = "~> 4.0"
    }

    sops = {
      source  = "carlpett/sops"
      version = "~> 0.5"
    }
  }

  backend "s3" {
    bucket         = "project-tfstate"
    dynamodb_table = "project-tfstate-lock"
    region         = "us-east-1"
    profile        = "project"
    encrypt        = true
  }
}

provider "aws" {
  region  = "eu-central-1"
  profile = "project"
}

provider "aws" {
  alias   = "us-east-1"
  region  = "us-east-1"
  profile = "project"
}

provider "cloudflare" {
  api_token = data.sops_file.secrets.data["cloudflare_api_token"]
}

data "cloudflare_zone" "domain" {
  name = "project.com"
}

data "sops_file" "secrets" {
  source_file = "secrets.enc.json"
}

module "frontend" {
  source = "github.com/bdeak4/terraform-module-s3-static-website"

  bucket_name        = "project-frontend-production"
  bucket_versioning  = true
  website_domain     = "project.com"
  cloudflare_zone_id = data.cloudflare_zone.domain.id

  tags = {
    Project     = "project"
    Role        = "frontend"
    Environment = "production"
  }

  providers = {
    aws.us-east-1 = aws.us-east-1
  }
}
```
