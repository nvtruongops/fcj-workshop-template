---
title: "CDK Bootstrap"
date: 2025-12-01
weight: 3
chapter: false
pre: " <b> 5.03. </b> "
---

#### Overview

AWS CDK Bootstrap prepares your AWS account for deploying CDK applications by creating essential infrastructure resources. This is a one-time setup process per AWS account/region combination that establishes the foundation for all CDK deployments.

#### What CDK Bootstrap Creates

The bootstrap process provisions the following AWS resources in your account:

**1. S3 Bucket (CDK Assets Bucket)**
- Stores Lambda function deployment packages
- Stores CloudFormation templates
- Stores file assets referenced by your CDK stacks
- Naming pattern: `cdk-hnb659fds-assets-{ACCOUNT-ID}-{REGION}`
- Versioning enabled for rollback support

**2. CloudFormation Stack (CDKToolkit)**
- Main infrastructure stack containing all bootstrap resources
- Stack name: `CDKToolkit`
- Manages lifecycle of bootstrap resources

**3. IAM Roles**
- `cdk-hnb659fds-cfn-exec-role`: CloudFormation execution role for deploying stacks
- `cdk-hnb659fds-deploy-role`: Role used by CDK CLI during deployment
- `cdk-hnb659fds-file-publishing-role`: Role for uploading assets to S3
- `cdk-hnb659fds-lookup-role`: Role for reading environment context

**4. SSM Parameters**
- `/cdk-bootstrap/hnb659fds/version`: Stores bootstrap version number
- Used for compatibility checking

#### Infrastructure Stacks Overview

After bootstrapping, our project will deploy multiple CDK stacks defined in `D:\Project_AWS\everyonecook\infrastructure\lib\stacks`:

- **auth-stack.ts**: Authentication and authorization infrastructure (Cognito)
- **backend-stack.ts**: Core backend services and Lambda functions
- **certificate-stack.ts**: SSL/TLS certificates management
- **core-stack.ts**: Core infrastructure resources
- **dns-stack.ts**: Route 53 DNS configuration
- **observability-stack.ts**: Monitoring, logging, and observability

These stacks depend on the bootstrap infrastructure to deploy successfully.

---

### Lab Instructions: Bootstrap Your AWS Account

#### Prerequisites

Before beginning this lab, ensure you have:

-  AWS CLI installed and configured (see section 5.1)
-  AWS CDK CLI installed (see section 5.2)
-  Valid AWS credentials configured
-  IAM user with `AdministratorAccess` permissions or equivalent
-  Terminal/PowerShell access to the project directory


### Step 3: Verify on AWS Console
Confirm resources in the AWS Management Console.
**3.1. Verify CloudFormation Stack**

1. Open [CloudFormation Console](https://console.aws.amazon.com/cloudformation/home?region=us-east-1)
2. Look for stack named `CDKToolkit`
3. Status should be `CREATE_COMPLETE`
4. Click on **Resources** tab to view created resources

** Screenshot Required:**
![CloudFormation CDKToolkit Stack](/images/5-Workshop/5.2-setup-environment/cloudformation-cdktoolkit.png)


**3.2. Verify S3 Bucket**

1. Open [S3 Console](https://s3.console.aws.amazon.com/s3/buckets?region=us-east-1)
2. Find bucket: `cdk-hnb659fds-assets-{ACCOUNT-ID}-us-east-1`
3. Verify bucket properties:
   - Versioning: Enabled

** Screenshot Required:**
![S3 Bucket Console](/images/5-Workshop/5.2-setup-environment/s3-bucket-console.png)


**3.3. Verify IAM Roles**

1. Open [IAM Roles Console](https://console.aws.amazon.com/iam/home#/roles)
2. Search for: `cdk-hnb659fds`
3. Verify the following roles exist:
   - `cdk-hnb659fds-cfn-exec-role-{ACCOUNT-ID}-us-east-1`
   - `cdk-hnb659fds-deploy-role-{ACCOUNT-ID}-us-east-1`
   - `cdk-hnb659fds-file-publishing-role-{ACCOUNT-ID}-us-east-1`
   - `cdk-hnb659fds-lookup-role-{ACCOUNT-ID}-us-east-1`

![IAM Roles Console](/images/5-Workshop/5.2-setup-environment/iam-roles-console.png)

---

### Understanding Bootstrap Resources

#### S3 Bucket Details

The CDK assets bucket serves as a central repository for all deployment artifacts:

- **Purpose**: Stores CloudFormation templates, Lambda deployment packages, and static assets
- **Lifecycle**: Persists across deployments, enabling rollback capabilities
- **Security**: Encrypted at rest, bucket policies restrict access to authorized roles
- **Naming**: Deterministic naming based on account ID and region


**1. CloudFormation Execution Role (`cfn-exec-role`)**
- Used by CloudFormation to create/update/delete stack resources
- Has broad permissions to manage AWS resources
- Trust relationship with CloudFormation service

**2. Deploy Role (`deploy-role`)**
- Used by CDK CLI during `cdk deploy` operations
- Can assume the CloudFormation execution role
- Has permissions to upload assets and initiate deployments

**3. File Publishing Role (`file-publishing-role`)**

**4. Image Publishing Role (`image-publishing-role`)**

**5. Lookup Role (`lookup-role`)**
- Reads environment context (VPCs, subnets, etc.)
- Read-only permissions for resource lookups
- Used during synthesis for context queries

---

### Cost Analysis

#### Bootstrap Resources Cost

**One-time Setup:**
- CloudFormation stack creation: **Free**
- IAM roles creation: **Free**
- SSM parameters: **Free**

**Ongoing Monthly Costs:**

| Resource | Usage | Cost (Estimated) |
|----------|-------|------------------|
| S3 Storage | < 1 GB (typical) | $0.023/GB = ~$0.02/month |
| S3 Requests | Minimal (GET/PUT) | ~$0.01/month |
| IAM Roles | No charge | $0.00 |

**Total Estimated Cost:** ~$0.03/month (negligible)

**Note:** For serverless applications, costs remain minimal as we only store Lambda deployment packages in S3.

---

### Lab Completion Checklist

Ensure you have completed all steps and captured required screenshots:

-  Step 1: Bootstrap command execution and success - **[IMAGE HERE: bootstrap-success.png]**
-  Step 2: CLI verification results - **[IMAGE HERE: bootstrap-verification-cli.png]**
-  Step 3.1: CloudFormation console - **[IMAGE HERE: cloudformation-cdktoolkit.png]**
-  Step 3.2: S3 bucket console - **[IMAGE HERE: s3-bucket-console.png]**
-  Step 3.3: IAM roles console - **[IMAGE HERE: iam-roles-console.png]**
1.  Verified AWS account bootstrap status
2.  Retrieved your AWS account ID
3.  Executed CDK bootstrap command
4.  Created CDKToolkit CloudFormation stack
5.  Provisioned CDK assets S3 bucket
6.  Created necessary IAM roles for CDK deployments
7.  Verified bootstrap version in SSM Parameter Store
8.  Understood the purpose and cost of each bootstrap resource
In this lab, you successfully:

1.  Executed CDK bootstrap command for your AWS account
2.  Created CDKToolkit CloudFormation stack
3.  Provisioned CDK assets S3 bucket for storing Lambda packages
4.  Created necessary IAM roles for CDK deployments
5.  Verified all resources via CLI and AWS Console

Your AWS account is now ready to deploy serverless CDK applications, including the infrastructure stacks for the EveryoneCook project.
- Prepare for infrastructure deployment
