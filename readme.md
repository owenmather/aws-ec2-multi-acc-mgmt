# AWS EC2 Multi Account Management

## Description

This repo serves as a PoC for managing EC2 instances in an AWS Organizations accross multiple accounts.  

## Global Prerequisites

The following resources are required to set up the **controller** environment  
1) GitHub Repo, GitHub Workflow and GitHub Secrets for running the Ansible Playbooks
2) AWS S3 Bucket for Ansible SSM Connection Plugin*
3) Example Ansible playbook to test PoC

> *The Ansible SSM Connection plugin uploads each Ansible Task as Python to a S3 Bucket. Then Ansible SSM Connection plugin then generates a presigned URL for S3 from the controller, and then will pass that URL to the target over SSM, telling the target to download/upload from S3 with curl.  
>   
> 1) Due to this design the Target EC2 Instance Profile does not need permissions on the S3 Bucket
> 2) For our PoC all accounts will use **one global** s3 bucket for transferring data/tasksm however the user is recommended to set up one bucket per region, at minimum, in production. For extra security the user could also consider deploying one s3 bucket per target AWS Account + Region in production. As the Ansible Tasks may transfer sensitive data to EC2 hosts and there is a _potential_ risk that this data may be left permanently in the transfer S3 Bucket (timeouts/networking or error conditions).
> 3) It is also recommended users add a s3 lifecycle policy, to delete any left over transfer data due to error conditions, to the S3 bucket(s) used for the above reason. We will consider this out of scope for this PoC 

## Target AWS Account Prerequisites

Each target AWS Accounts needs the following resources:  
1) AWS Role with a **fixed** naming convention. For this PoC we will use the name: `oidc-org-ec2-management-role`  
2) Trusted Policy Configured on the Role to allow the GitHub repo to assume the role with **oidc**  
3) Permissions on the role to connect to ec2 instances via SSM and describe instances in the AWS Account  
4) Permissions for the role to write data to Global S3 Bucket

## Deployment Preparation  

From this point in the document we will use the following variable names for reference - the user should decide on their own names/naming conventions for their org and update the usages appropriately:

| Variable           | Example Value                  | Resource    | Details                                              |
|--------------------|--------------------------------|-------------|------------------------------------------------------|
| `$S3_BUCKET`       | `ssm_s3_transfer_bucket`       | S3 Bucket   | The global S3 transfer bucket name                   |
| `$OIDC_ROLE_NAME`  | `oidc-org-ec2-management-role` | IAM Role    | The IAM Role name we will assume in each AWS Account |
| `$AWS_ACCOUNT_IDS` | `[ AAAAAAAA, BBBBBBBB ]`       | AWS Account | The list of Target AWS Accounts for this PoC         |
| `$GITHUB_REPO`     | `aws-ec2-multi-acc-mgmaws-ec2-multi-acc-mgmt`       | GitHub Repo | The GitHub Repo for the Controller Environment       |

## Global Setup Instructions

### 1) GitHub Repo, GitHub Workflow and GitHub Secrets for running the Ansible Playbooks

The user should create a **private** GitHub repo for the PoC in their GitHub Org/Acc  
The user should copy all files in this repo to theirs (can omit this readme if required)  

The user must set values for the account(s) and region(s) targeted in [ansible_run.yml](.github/workflows/ansible_run.yml)  

![img.png](images/img.png)

The user must replace the value for `$OIDC_ROLE_NAME` in [ansible_run.yml](.github/workflows/ansible_run.yml)  
![img_1.png](images/img_1.png)
**:bangbang This must be the same name in all target accounts!**

### 1) AWS S3 Bucket for Ansible SSM Connection Plugin*

### 2) Example Ansible playbook to test PoC

### 4: Permissions for the role to write data to Global S3 Bucket

The controller requires IAM permissions to upload, download and delete files from the specified S3 bucket. This includes `s3:GetObject`, `s3:PutObject`, `s3:ListBucket`, `s3:DeleteObject` and `s3:GetBucketLocation`.


## Target AWS Account Setup Instructions
