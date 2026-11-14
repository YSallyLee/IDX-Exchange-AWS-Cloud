# Week 08 — Infrastructure as Code (Terraform)

## 1. Overview
This week transitioned infrastructure provisioning from manual AWS Console clicks to **Infrastructure as Code (IaC)** using **Terraform**. Using the official `terraform-aws-modules/vpc/aws` module, I reproduced Week 5's multi-AZ custom VPC (`training-vpc-tf`, CIDR `10.1.0.0/16`)—complete with 2 public subnets, 2 private subnets, an Internet Gateway, a single NAT Gateway, and route table associations—in a single declarative configuration (`main.tf`).

---

## 2. Core Terraform Lifecycle Commands

### `terraform plan` (Dry-Run & Execution Preview)
* **What it does**: Reads the desired state defined in `.tf` configuration files, queries the live AWS environment, and compares both against the local `terraform.tfstate` file. It outputs an execution plan showing exactly which resources will be added (`+`), modified (`~`), or destroyed (`-`) without making any actual changes to cloud infrastructure.
* **Why it matters**: Acts as a critical safety check before deployment, preventing unintended modifications or accidental deletion of production resources[cite: 3].

### `terraform apply` (Provisioning & Reconciliation)
* **What it does**: Executes the actions proposed in the execution plan by calling AWS APIs to create, update, or delete resources until the real-world cloud infrastructure matches the declarative code[cite: 3]. Once complete, it writes the updated resource IDs and attributes into `terraform.tfstate`.
* **Idempotency**: Running `terraform apply` repeatedly when no code has changed results in zero changes, guaranteeing consistent environments across dev, staging, and production.

### `terraform destroy` (Full Environment Teardown)
* **What it does**: Reads `terraform.tfstate` to identify all resources managed by the configuration, resolves reverse dependency order (e.g., deleting the NAT Gateway and releasing the Elastic IP before deleting subnets and the VPC), and tears down the entire stack cleanly.
* **Why it matters**: Eliminates orphaned cloud resources and prevents recurring hourly charges (such as NAT Gateway costs) once testing is complete[cite: 3].

---

## 3. Why Terraform State (`terraform.tfstate`) Matters

1. **Mapping Code to Real Cloud Resources**: `terraform.tfstate` is a JSON file that tracks the exact binding between logical resource blocks in `main.tf` and their physical AWS Resource IDs (e.g., `vpc-051484d7e2c654358`). Without state, Terraform would not know which resources it owns or manages.
2. **Drift Detection (Debug Lab 8.2)**: When someone bypasses Terraform and manually edits a resource in the AWS Console (simulated by manually changing the VPC `Name` tag to `training-vpc-tf-DRIFT`), `terraform plan` detects the discrepancy between the live cloud attribute and the state/desired configuration, allowing `terraform apply` to reconcile and restore the intended state[cite: 3].
3. **Security & `.gitignore` Hygiene**: Because state files store resource metadata and sensitive attributes in plain text, `.terraform/` and `*.tfstate*` must always be excluded via `.gitignore` and never committed to public Git repositories.

---

## 4. Cleanup & Cost Verification
* [x] Executed `terraform destroy` to remove all 19 provisioned network resources (`Destroy complete! Resources: 19 destroyed.`).
* [x] Verified in the AWS Console that `training-vpc-tf` and its NAT Gateway / Elastic IP are completely removed.
* [x] Checked AWS Billing Dashboard (`billing-screenshot.png`) to confirm no unexpected charges[cite: 3].