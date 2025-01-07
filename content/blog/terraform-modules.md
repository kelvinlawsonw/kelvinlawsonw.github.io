+++
title = 'Understanding Terraform Modules'
date = 2025-01-05T21:56:26-05:00
draft = false
tags = ["terraform", "iac", "modules" ]
+++


### What are Terraform Modules 

A Terraform module in it's simplest form is a container for multiple resources that are used together. This is also a direct definition from the __[Hashicorp website](https://developer.hashicorp.com/terraform/language/modules/develop)__. It’s a logical grouping of resources that can be reused across different parts of your infrastructure, helping to reduce duplication and increase ease of maintainance.

There are typically two types of modules:
+ Root Module: This is the top level configuration in your working directory where the resources for your primary infrastructure are defined.
+ Child Module: This is called by another module using the module block and can be reused as many times as required.

A typical module is structured just as follows:

```
├── main.tf
├── outputs.tf
├── README.md (optional)
└── variables.tf
└── providers.tf (optional)
```

Our scenario requires the creation of a simple S3 bucket. This will allow us illustrate how a module is created and it's usage:

### main.tf
```
resource "aws_s3_bucket" "this" {
  bucket        = var.bucket_name
  acl           = var.acl
  force_destroy = var.force_destroy

  tags = var.tags
}
```

### variables.tf
```
variable "bucket_name" {
  description = "The name of the S3 bucket"
  type        = string
}

variable "acl" {
  description = "The access control list (ACL) for the S3 bucket"
  type        = string
  default     = "private"
}

variable "force_destroy" {
  description = "Whether to force destroy the bucket when it is deleted"
  type        = bool
  default     = false
}

variable "tags" {
  description = "Tags to apply to the S3 bucket"
  type        = map(string)
  default     = {}
}
```

### outputs.tf
```
output "bucket_id" {
  description = "The ID of the S3 bucket"
  value       = aws_s3_bucket.this.id
}

output "bucket_arn" {
  description = "The ARN of the S3 bucket"
  value       = aws_s3_bucket.this.arn
}
```

### Example Usage

The module can now be called and used as many times as required.

```
module "s3_bucket" {
  source        = "./path/to/your/module"
  bucket_name   = "my-awesome-new-bucket"
  acl           = "private"
  force_destroy = true
  tags = {
    Owner       = "FirstName LastName"
  }
}
```
### Benefits of Terraform Modules

+ Reusability: Modules promote code-reusability by allowing engineers to write once and use across multiple projects and environments. Modules also help adherence to the DRY (Don't Repeat Yourself) principle by abrastracting repetive patterns.

+ Consistency: Modules ensure that infrastructure follows the same pattern reducing human error and avoiding any discrepancies.

+ Collaboration: Teams of any size can collaborate better by using shared modules.

+ Scalability: The task of scaling infrastructure is made easier as this modular approach can be used to deploy identical resources in different regions.

