# IAM & Least Privilege Enforcement (Unit 2)

Terraform project that builds a least-privilege IAM structure:

- **IAM Groups**: `Admins`, `Developers`, `Auditors`, each with a distinct custom policy scoped to what that role actually needs.
- **MFA enforcement**: An additional policy attached to the `Admins` group denies almost every action unless the caller has authenticated with MFA (`aws:MultiFactorAuthPresent`), while still letting users enroll/manage their own MFA device so they don't get locked out.
- **EC2 → S3 access without hardcoded credentials**: A custom IAM Role (`EC2-S3-Access-Role`) + instance profile that an EC2 instance can assume to read/write a specific S3 bucket, with no access keys stored anywhere.

## Files

| File | Purpose |
|---|---|
| `main.tf` | All IAM groups, policies, role, and instance profile |
| `variables.tf` | Input variables (region, bucket name) |
| `outputs.tf` | Useful outputs after apply |
| `terraform.tfvars.example` | Copy to `terraform.tfvars` and fill in your own bucket name |

## How the least-privilege design works

- **Admins** get broad operational access (EC2, S3, IAM group/role management, CloudWatch/Logs) but are explicitly **denied** billing, AWS Organizations, and account-level actions — so even the "admin" tier isn't root-equivalent.
- **Developers** can run/stop/terminate EC2 instances and read/write only the app S3 bucket. IAM actions are explicitly denied.
- **Auditors** get read-only (`Get*`, `List*`, `Describe*`) access across IAM, S3, EC2, CloudTrail, and Config — enough to audit, nothing that can change state.
- The **EC2 role**'s policy is scoped to just the one bucket's ARN (and its objects), not `"*"`, so a compromised instance can't reach unrelated buckets.

## Deploy it yourself

```bash
git clone <your-repo-url>
cd iam-least-privilege-project
cp terraform.tfvars.example terraform.tfvars
# edit terraform.tfvars with a globally-unique bucket name

terraform init
terraform plan
terraform apply
```

You'll need AWS credentials configured locally (`aws configure` or environment variables) with permission to create IAM and S3 resources. This project does **not** need to be run by me — it's meant to be applied in your own AWS account/sandbox so you get real console output for your screenshot.

## Getting your screenshot

After `terraform apply` succeeds, good screenshot options for the assignment:
- The AWS Console → IAM → Groups, showing `Admins`, `Developers`, `Auditors` with their attached policies.
- IAM → Roles → `EC2-S3-Access-Role`, showing the trust relationship (`ec2.amazonaws.com`) and attached policy.
- Terminal output of `terraform apply` showing all resources created successfully.
- (Optional, to prove MFA enforcement) An Admins-group IAM user attempting an S3 action without MFA and getting an `AccessDenied`.

## Cleanup

```bash
terraform destroy
```
