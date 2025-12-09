---
title: "Cleanup"
date: 2025-12-01
weight: 11
chapter: false
pre: " <b> 5.11. </b> "
---

#### Overview

When you complete the workshop and no longer want to continue using it, delete all resources to avoid incurring charges.

**⚠️ Warning:** This process cannot be undone. All data will be permanently deleted.

#### Cleanup Order

Must delete in reverse order of deployment:

```
1. Amplify App (Frontend)
2. Observability Stack
3. Backend Stack
4. Auth Stack
5. Core Stack
6. Certificate Stack
7. DNS Stack
8. CDK Bootstrap (optional)
```

#### Step 1: Delete Amplify App

**1. Delete via Console**

1. Go to AWS Amplify Console
2. Select your app
3. Click "Actions" → "Delete app"
4. Type app name to confirm
5. Click "Delete"

**2. Delete via CLI**

```bash
# Get app ID
APP_ID=$(aws amplify list-apps \
  --query 'apps[?name==`everyonecook-frontend`].appId' \
  --output text)

# Delete app
aws amplify delete-app --app-id $APP_ID
```

#### Step 2: Empty S3 Buckets

**S3 buckets must be emptied before deletion:**

```bash
# List all EveryoneCook buckets
aws s3 ls | grep everyonecook

# Empty each bucket (4 buckets)
aws s3 rm s3://everyonecook-content-dev-xxxxx --recursive
aws s3 rm s3://everyonecook-logs-dev-xxxxx --recursive
aws s3 rm s3://everyonecook-emails-dev-xxxxx --recursive
aws s3 rm s3://everyonecook-cdn-logs-dev-xxxxx --recursive

# Or use PowerShell script
$buckets = aws s3 ls | Select-String "everyonecook" | ForEach-Object { $_.ToString().Split()[-1] }
foreach ($bucket in $buckets) {
  Write-Host "Emptying bucket: $bucket"
  aws s3 rm "s3://$bucket" --recursive
}
```

#### Step 3: Delete CDK Stacks

**1. Delete Observability Stack**

```bash
cd infrastructure

# Delete Observability stack
npx cdk destroy EveryoneCook-dev-Observability --context environment=dev

# Type 'y' to confirm
```

**2. Delete Backend Stack**

```bash
# Delete Backend stack
npx cdk destroy EveryoneCook-dev-Backend --context environment=dev

# Type 'y' to confirm
```

**3. Delete Auth Stack**

```bash
# Delete Auth stack
npx cdk destroy EveryoneCook-dev-Auth --context environment=dev

# Type 'y' to confirm
```

**4. Delete Core Stack**

```bash
# Delete Core stack (takes 15-20 minutes due to CloudFront)
npx cdk destroy EveryoneCook-dev-Core --context environment=dev

# Type 'y' to confirm
```

**5. Delete Certificate Stack**

```bash
# Delete Certificate stack
npx cdk destroy EveryoneCook-dev-Certificate --context environment=dev

# Type 'y' to confirm
```

**6. Delete DNS Stack (Optional)**

```bash
# Delete DNS stack (only if you don't need the domain)
npx cdk destroy EveryoneCook-dev-DNS --context environment=dev

# Type 'y' to confirm
```

#### Step 4: Verify All Stacks Deleted

```bash
# List remaining stacks
aws cloudformation list-stacks \
  --stack-status-filter CREATE_COMPLETE UPDATE_COMPLETE \
  --query 'StackSummaries[?contains(StackName, `EveryoneCook-dev`)].StackName'

# Should return empty array
```

#### Step 5: Delete CDK Bootstrap (Optional)

**⚠️ Only do this if you're done with CDK completely:**

```bash
# Delete CDK bootstrap stack
aws cloudformation delete-stack --stack-name CDKToolkit

# Empty and delete CDK assets bucket
BUCKET_NAME=$(aws s3 ls | grep cdk | awk '{print $3}')
aws s3 rm s3://$BUCKET_NAME --recursive
aws s3 rb s3://$BUCKET_NAME
```

#### Step 6: Delete GitLab Repository (Optional)

**1. Archive Repository**

1. Go to GitLab → Settings → General
2. Scroll to "Advanced"
3. Click "Archive project"

**2. Delete Repository**

1. Go to GitLab → Settings → General
2. Scroll to "Advanced"
3. Click "Delete project"
4. Type project name to confirm
5. Click "Yes, delete project"

#### Step 7: Verify Complete Cleanup

**1. Check CloudFormation**

```bash
aws cloudformation list-stacks \
  --stack-status-filter CREATE_COMPLETE UPDATE_COMPLETE \
  | grep EveryoneCook

# Should return nothing
```

**2. Check S3**

```bash
aws s3 ls | grep everyonecook

# Should return nothing
```

**3. Check Lambda**

```bash
aws lambda list-functions | grep EveryoneCook

# Should return nothing
```

**4. Check DynamoDB**

```bash
aws dynamodb list-tables | grep EveryoneCook

# Should return nothing
```

**5. Check API Gateway**

```bash
aws apigateway get-rest-apis | grep EveryoneCook

# Should return nothing
```

**6. Check Cognito**

```bash
aws cognito-idp list-user-pools --max-results 10 | grep EveryoneCook

# Should return nothing
```

**7. Check CloudFront**

```bash
aws cloudfront list-distributions | grep EveryoneCook

# Should return nothing
```

**8. Check OpenSearch**

```bash
aws opensearch list-domain-names | grep everyonecook

# Should return nothing
```

**9. Check Amplify**

```bash
aws amplify list-apps | grep everyonecook

# Should return nothing
```

#### Cost After Cleanup

**Immediate:**
- Most resources: $0/month
- Route 53 Hosted Zone: $0.50/month (if kept)
- KMS keys: Scheduled for deletion (7-30 days), no charge during waiting period

**After 30 days:**
- Everything: $0/month (if DNS stack also deleted)

#### Troubleshooting Cleanup

**Issue: S3 bucket deletion fails**

```bash
# Force empty and delete
aws s3 rb s3://bucket-name --force
```

**Issue: CloudFormation stack stuck**

```bash
# Check stack events
aws cloudformation describe-stack-events \
  --stack-name EveryoneCook-dev-Core \
  --max-items 20

# If stuck, skip failed resources
aws cloudformation delete-stack \
  --stack-name EveryoneCook-dev-Core \
  --retain-resources ResourceLogicalId
```

**Issue: CloudFront distribution can't be deleted**

```bash
# Disable distribution first
DIST_ID=$(aws cloudfront list-distributions \
  --query "DistributionList.Items[?Comment=='EveryoneCook-dev'].Id" \
  --output text)

# Get ETag
ETAG=$(aws cloudfront get-distribution --id $DIST_ID \
  --query "ETag" --output text)

# Disable distribution
aws cloudfront update-distribution \
  --id $DIST_ID \
  --if-match $ETAG \
  --distribution-config file://disabled-config.json

# Wait 15-20 minutes, then delete
aws cloudfront delete-distribution --id $DIST_ID --if-match $ETAG
```

**Issue: DynamoDB table has deletion protection**

```bash
# Disable deletion protection
aws dynamodb update-table \
  --table-name EveryoneCook-dev \
  --no-deletion-protection-enabled

# Then delete stack
```

#### Final Verification

```bash
# Check AWS billing
aws ce get-cost-and-usage \
  --time-period Start=2024-01-01,End=2024-01-31 \
  --granularity MONTHLY \
  --metrics BlendedCost

# Should show decreasing costs
```

#### Cleanup Checklist

- [ ] Amplify app deleted
- [ ] All S3 buckets emptied and deleted
- [ ] Observability stack deleted
- [ ] Backend stack deleted
- [ ] Auth stack deleted
- [ ] Core stack deleted
- [ ] Certificate stack deleted
- [ ] DNS stack deleted (optional)
- [ ] CDK bootstrap deleted (optional)
- [ ] GitLab repository archived/deleted (optional)
- [ ] No remaining CloudFormation stacks
- [ ] No remaining Lambda functions
- [ ] No remaining DynamoDB tables
- [ ] No remaining S3 buckets
- [ ] No remaining CloudFront distributions
- [ ] No remaining Cognito user pools
- [ ] Billing shows $0 or minimal cost

#### Conclusion

You have completed the EveryoneCook workshop! You have learned:

 **Infrastructure as Code** with AWS CDK  
 **Serverless Architecture** with Lambda and API Gateway  
 **DynamoDB Single Table Design** with username-based PK  
 **OpenSearch** with Vietnamese analyzer  
 **CloudFront CDN** with Origin Access Control  
 **Cognito Authentication** with Lambda triggers  
 **Bedrock AI** integration  
 **GitLab** version control and CI/CD  
 **AWS Amplify** frontend deployment  
 **CloudWatch** monitoring and X-Ray tracing

**Total deployed:**
- 7 CDK stacks
- 100+ AWS resources
- 6 Lambda modules + 1 worker
- Full-stack application
- CI/CD pipeline
- Production-ready infrastructure

Thank you for completing the workshop! 🎉
