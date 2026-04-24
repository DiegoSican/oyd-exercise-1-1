# Exercise 1.1 — What Will Terraform Do?

**Course:** Optimizaciones y Desempeño — Cloud Deployment Automation  
**Session:** 1 — April 23, 2026  
**Student:** Diego Sican  

---

## Objective

The objective of this exercise is to analyze a Terraform configuration and understand what Terraform plans to do **without running `terraform apply`**.

The configuration defines an AWS S3 bucket named:

```hcl
oyd-exercise-bucket-2026
```

The exercise focuses on reading Terraform output, understanding the plan/apply boundary, and reasoning about Terraform state.

---

# Task 1 — Initialize and Validate

## Commands executed

```bash
terraform init
terraform validate
```

## `terraform init` output

```bash
Initializing the backend...
Initializing provider plugins...
- Reusing previous version of hashicorp/aws from the dependency lock file
- Using previously-installed hashicorp/aws v5.100.0

Terraform has been successfully initialized!

You may now begin working with Terraform. Try running "terraform plan" to see
any changes that are required for your infrastructure. All Terraform commands
should now work.

If you ever set or change modules or backend configuration for Terraform,
rerun this command to reinitialize your working directory. If you forget, other
commands will detect it and remind you to do so if necessary.
```

## `terraform validate` output

```bash
Success! The configuration is valid.
```

## Result

The Terraform configuration was successfully initialized and validated. No syntax or configuration errors were found.

---

# Task 2 — Read the Plan

## Command executed

```bash
terraform plan
```

## Full `terraform plan` output

> Paste the full output of the first `terraform plan` here.

```bash
    # aws_s3_bucket.exercise will be created
      + resource "aws_s3_bucket" "exercise" {
          + acceleration_status         = (known after apply)
          + acl                         = (known after apply)
          + arn                         = (known after apply)
          + bucket                      = "oyd-exercise-bucket-2026"
          + bucket_domain_name          = (known after apply)
          + bucket_prefix               = (known after apply)
          + bucket_regional_domain_name = (known after apply)
          + force_destroy               = false
          + hosted_zone_id              = (known after apply)
          + id                          = (known after apply)
          + object_lock_enabled         = (known after apply)
          + policy                      = (known after apply)
          + region                      = (known after apply)
          + request_payer               = (known after apply)
          + tags                        = {
              + "Environment" = "dev"
              + "ManagedBy"   = "terraform"
              + "Owner"       = "Diego Sican"
            }
          + tags_all                    = {
              + "Environment" = "dev"
              + "ManagedBy"   = "terraform"
              + "Owner"       = "Diego Sican"
            }
          + website_domain              = (known after apply)
          + website_endpoint            = (known after apply)

          + cors_rule (known after apply)

          + grant (known after apply)

          + lifecycle_rule (known after apply)

          + logging (known after apply)

          + object_lock_configuration (known after apply)

          + replication_configuration (known after apply)

          + server_side_encryption_configuration (known after apply)

          + versioning (known after apply)

          + website (known after apply)
        }
```

## Questions

### 1. How many resources will be created, changed, or destroyed?

Terraform proposed the following actions:

```text
Plan: 1 to add, 0 to change, 0 to destroy.
```

This means:

| Action | Quantity |
|---|---:|
| Create | 1 |
| Change | 0 |
| Destroy | 0 |

Terraform plans to create one AWS S3 bucket.

---

### 2. What does the `+` symbol next to each attribute mean?

The `+` symbol means that Terraform plans to **add or create** that resource or attribute.

For example:

```hcl
+ bucket = "oyd-exercise-bucket-2026"
```

This means Terraform plans to create the S3 bucket with that name.

---

### 3. Find one attribute in the plan marked as `(known after apply)`. Why can't Terraform know that value before apply?

One example is:

```hcl
+ arn = (known after apply)
```

Terraform cannot know this value before `apply` because the value is generated or confirmed by AWS only after the resource is actually created.

Since `terraform plan` only previews the actions and does not create the resource, Terraform marks some values as `(known after apply)`.

---

# Task 3 — Predict a Change

A new tag was added to the `tags` block:

```hcl
Owner = "Diego Sican"
```

The updated block is:

```hcl
tags = {
  Environment = "dev"
  ManagedBy   = "terraform"
  Owner       = "Diego Sican"
}
```

## Command executed

```bash
terraform plan
```

## Diff section only

```hcl
# aws_s3_bucket.exercise will be created
+ resource "aws_s3_bucket" "exercise" {
    + acceleration_status         = (known after apply)
    + acl                         = (known after apply)
    + arn                         = (known after apply)
    + bucket                      = "oyd-exercise-bucket-2026"
    + bucket_domain_name          = (known after apply)
    + bucket_prefix               = (known after apply)
    + bucket_regional_domain_name = (known after apply)
    + force_destroy               = false
    + hosted_zone_id              = (known after apply)
    + id                          = (known after apply)
    + object_lock_enabled         = (known after apply)
    + policy                      = (known after apply)
    + region                      = (known after apply)
    + request_payer               = (known after apply)
    + tags                        = {
        + "Environment" = "dev"
        + "ManagedBy"   = "terraform"
        + "Owner"       = "Diego Sican"
      }
    + tags_all                    = {
        + "Environment" = "dev"
        + "ManagedBy"   = "terraform"
        + "Owner"       = "Diego Sican"
      }
    + website_domain              = (known after apply)
    + website_endpoint            = (known after apply)

    + cors_rule (known after apply)

    + grant (known after apply)

    + lifecycle_rule (known after apply)

    + logging (known after apply)

    + object_lock_configuration (known after apply)

    + replication_configuration (known after apply)

    + server_side_encryption_configuration (known after apply)

    + versioning (known after apply)

    + website (known after apply)
  }
```

## Questions

### 1. Did Terraform propose to destroy and recreate the bucket, or update it in place?

Terraform did not propose to destroy and recreate the bucket, and it also did not propose an in-place update.

Since `terraform apply` has not been executed, Terraform does not have an existing state for this bucket. Therefore, Terraform still proposes to create the bucket, now including the new `Owner` tag.

---

### 2. Why does that distinction matter?

This distinction matters because destroying and recreating a resource can cause data loss, downtime, or changes in the resource identity.

An in-place update is safer because it modifies the existing resource without replacing it. In this case, because the resource has not been applied yet, Terraform only shows the full creation plan.

---

# Task 4 — State Reasoning

## Command executed

```bash
ls -la
```

On PowerShell, the equivalent command used was:

```powershell
dir -Force
```

## Output

```powershell
Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
d-----        23/04/2026     19:15                .terraform
-a----        23/04/2026     20:40            129 .gitignore
-a----        23/04/2026     19:15           1407 .terraform.lock.hcl
-a----        23/04/2026     19:58            379 main.tf
```

## Questions

### 1. Does a `.tfstate` file exist in this directory?

No, a `terraform.tfstate` file does not exist in this directory.

The `.terraform` directory and the `.terraform.lock.hcl` file exist because `terraform init` was executed. However, those files are not the Terraform state file.

---

### 2. What does the absence of the state file tell you about the relationship between plan and state?

The absence of the `terraform.tfstate` file shows that `terraform plan` does not create or update Terraform state.

The `plan` command only previews what Terraform would do by comparing the configuration with the known state. The state file is created or updated when `terraform apply` is executed.

This demonstrates the boundary between `plan` and `apply`:

| Command | Purpose |
|---|---|
| `terraform plan` | Previews the actions Terraform would take |
| `terraform apply` | Executes the actions and records them in the state file |

---

# Final Notes

No `terraform apply` command was executed during this exercise.

This repository contains:

```text
main.tf
.terraform.lock.hcl
.gitignore
README.md
```

The `.terraform/` directory and Terraform state files are ignored because they should not be committed to the repository.
