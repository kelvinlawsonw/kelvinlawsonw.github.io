+++
title = 'Terraform Test Framework: Say Goodbye to “Terraform Apply and Pray”'
date = 2025-01-05T21:56:26-05:00
draft = false
tags = ["terraform", "iac", "testing", "unit-tests",  ]
+++


### The Terraform Test Framework

Introduced in Terraform v1.6.x, The Terraform Test Framework was created to help validate infrastructure configurations before applying into any production environment. The testing framework introduces a native and built in way to define tests that you would like terraform to run when you want to implement either unit or integration testing.

In this post, we'll dive into:
+ The importance of the Terraform Test Framework
+ How the Terraform's built-in test framework works
+ A quick sample of writing and running Terraform Tests 

### The Importance of the Terraform Test Framework
Before it's introduction, Infrastructure and DevOps engineers had to rely on tools like Terratest or manual verification for testing. While these work, they add a layer of consideration with complexities due to additional tools, extra dependencies and costs or just the plain old human error. The Terraform Test Framework alleviates all issues by:
+ Enabling tests to be written in HCL just like your Terraform code.
+ Prevent any misconfigurations before any deployment by providing both unit and integration testing.
+ Supporting integration into CI/CD workflows.

### How the Framework Works 
The framework uses special tests files __(.tftest.hcl)__ that define the Test Setup, Assertions and Terraform Commands. Test configurations are written in HCL as previously mentioned and multiple .tftest.hcl files can exist in the test folder. These test files are executed alphabetically by filename. Each test configuration in a file has a provider block, a variables block, and one or more run blocks.

A run block may contain one or several {  assert } blocks with every {  assert } block evaluating to true for the run block to pass. 

It is important to always remember that all test files should use the __.tftest.hcl__ file ending or else the __terraform test__ command will not execute as required.

A sample directory structure with a test folder looks like this:

```
├── main.tf
├── outputs.tf
├── providers.tf
├── tests
│   └── my_test.tftest.hcl
├── version.tf
└── variables.tf
```

### Terraform Test Sample 

This sample walks through a simple scenario where we would like to test that our S3 bucket was created and assigned the desired name 

__Step 1: Create a simple module that creates an AWS S3 bucket__

```
provider "aws" {
  region = "us-east-1"
}

resource "aws_s3_bucket" "example" {
  bucket = var.bucket_name
}

variable "bucket_name" {
  type    = string
  default = "my-super-awesome-bucket"
}

output "bucket_name" {
  value = aws_s3_bucket.example.id
}
```

__Step 2: Create a Terraform Test File__

In our folder structure, this should reside in tests/my_test.tftest.hcl

```
run "s3_bucket_test" {

  command = apply

  assert {
    condition = output.bucket_name == "my-super-awesome-bucket"
    error_message = "Bucket name does not match expected value"
  }
}
```

__Step 3: Run the Test__

```
tests/my_test.tftest.hcl... in progress
  run "s3_bucket_test"...   pass
tests/my_test.tftest.hcl... tearing down
tests/my_test.tftest.hcl... pass

Success! 1 passed, 0 failed.

```

The Terraform Test runs the assertion to check the name of the bucket matches the provided value and passes the test. It also destroys the resources after a completed test.