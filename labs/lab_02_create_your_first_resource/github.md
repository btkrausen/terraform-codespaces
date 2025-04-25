# LAB-02-GH: Creating Your First GitHub Resource

## Overview
In this lab, you will create your first GitHub resources using Terraform: a repository and branch protection rules. We will build upon the configuration files created in LAB-01, adding resource configuration and implementing the full Terraform workflow. The lab introduces environment variables for GitHub authentication, resource blocks, and the essential Terraform commands for resource management.

[![Lab 02](https://github.com/btkrausen/terraform-testing/actions/workflows/github_lab_validation.yml/badge.svg?branch=main)](https://github.com/btkrausen/terraform-testing/actions/workflows/github_lab_validation.yml)

**Preview Mode**: Use `Cmd/Ctrl + Shift + V` in VSCode to see a nicely formatted version of this lab!

## Prerequisites
- Terraform installed
- GitHub account with appropriate permissions
- GitHub Personal Access Token (with repo and admin:org permissions)
- Completion of [LAB-01-GH](https://github.com/btkrausen/terraform-codespaces/blob/main/labs/lab_01_getting_started_with_terraform/github.md)

## Estimated Time
20 minutes

## Lab Steps

### 1. Navigate to Your Configuration Directory

Ensure you're in the terraform directory created in LAB-01:

```bash
pwd
/workspaces/terraform-codespaces/labs/terraform
```
If you're in a different directory, change to the Terraform working directory:
```bash
cd labs/terraform
```

### 2. Configure GitHub Credentials

Set your GitHub credentials as environment variables:

```bash
export GITHUB_TOKEN="your_personal_access_token"
```

### 3. Add Resource Configuration

Open `main.tf` and add the following configuration (purposely not written in HCL canonical style):

```hcl
# Create the repository
resource "github_repository" "example" {
  name = "terraform-example"
  description = "Repository created by Terraform"
  visibility = "public"

  auto_init = true

  has_issues = true
  has_discussions = true
  has_wiki = true

  allow_merge_commit = true
  allow_squash_merge = true
  allow_rebase_merge = true
  
  topics = ["terraform", "infrastructure-as-code"]
}

# Create branch protection rule
resource "github_branch_protection" "main" {
  repository_id = github_repository.example.node_id
  pattern       = "main"

  required_pull_request_reviews {
    required_approving_review_count = 1
  }
}
```

### 4. Format and Validate

Format your configuration to rewrite it to follow HCL style:
```bash
terraform fmt
```

Validate the syntax:
```bash
terraform validate
```

### 5. Review the Plan

Generate and review the execution plan:
```bash
terraform plan
```

The plan output will show that Terraform intends to create:
- A new private repository with specified features
- A branch protection rule requiring one review for the main branch

### 6. Apply the Configuration

Apply the configuration to create the resources:
```bash
terraform apply
```

Review the proposed changes and type `yes` when prompted to confirm.

### 7. Verify the Resources

Let's verify our resources in the GitHub web interface:

1. Open your web browser and navigate to `GitHub.com`
2. Go to your repositories list
3. You should see the new `terraform-example` repository
4. Click into the repository to verify:
   - The repository description
   - The enabled features (Issues, Discussions, Wiki)
   - The repository topics
5. Navigate to Settings → Branches to verify:
   - The branch protection rule is applied to the main branch
   - Pull request reviews are required

### 8. Update the Repository Settings

In the `main.tf` file, update the repository configuration:

```hcl
resource "github_repository" "example" {
  name        = "terraform-example"
  description = "Updated repository description" # <-- change description
  visibility  = "public"

  auto_init = true

  has_issues      = true
  has_discussions = true
  has_wiki        = false # <-- change wiki setting

  allow_merge_commit = true
  allow_squash_merge = true
  allow_rebase_merge = true

  topics = ["terraform", "infrastructure-as-code", "learning"] # <-- add topic
}
```

### 9. Run a Terraform Plan to Perform a Dry Run

Generate and review the execution plan:
```bash
terraform plan
```

The plan output will show that Terraform will update the repository in-place:
- The description will be updated
- Wiki feature will be disabled
- A new topic will be added

### 10. Apply the Configuration

Apply the configuration to update the repository:
```bash
terraform apply
```

Review the proposed changes and type `yes` when prompted to confirm.

### 11. Update the Branch Protection

In the `main.tf` file, update the branch protection configuration:

```hcl
resource "github_branch_protection" "main" {
  repository_id = github_repository.example.node_id
  pattern       = "main"

  required_pull_request_reviews {
    required_approving_review_count = 2  # <-- increase required reviewers
  }
}
```

### 12. Run a Terraform Plan to Perform a Dry Run

Generate and review the execution plan:
```bash
terraform plan
```

The plan output will show that Terraform will update the branch protection rule:
- Required reviewers will be increased to `2`

### 13. Apply the Configuration

Apply the configuration to update the branch protection:
```bash
terraform apply
```

Review the proposed changes and type `yes` when prompted to confirm.

## Verification Steps

Confirm that:
1. The resources exist in your GitHub account with:
   - Repository named `terraform-course-repo`
   - Updated description and topics
   - Wiki disabled
   - Branch protection requiring `2` reviewers
2. A `terraform.tfstate` file exists in your directory
3. All Terraform commands completed successfully

## Success Criteria
Your lab is successful if:
- GitHub credentials are properly configured using environment variables
- The resources are successfully created with all specified configurations
- All Terraform commands execute without errors
- The `terraform.tfstate` file accurately reflects your infrastructure
- The resources are successfully destroyed during cleanup

## Additional Exercises
1. Try changing other repository settings (e.g., merge options)
2. Add additional branch protection rules
3. Review the `terraform.tfstate` file to understand how Terraform tracks resource state

## Common Issues and Solutions

If you encounter credential errors:
- Double-check your personal access token
- Ensure the token has the required permissions
- Verify the token hasn't expired

If you see permission issues:
- Verify you have admin access to the repository
- Check that your token includes required scopes
- Ensure you're not hitting repository limits for your account type

## Next Steps
In the next lab, we will build upon this by adding repository collaborators and additional settings. Keep your Terraform configuration files intact, as we will continue to expand upon them.