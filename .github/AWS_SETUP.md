# AWS Setup for GitHub Actions OIDC Authentication

This guide explains how to set up AWS to allow GitHub Actions to deploy to S3 using OIDC (OpenID Connect) federated authentication.

## Prerequisites

- AWS CLI configured with admin access
- Your GitHub repository (e.g., `raaedserag/elgasos.serag.net`)

## Step 1: Create the OIDC Identity Provider

```bash
aws iam create-open-id-connect-provider \
  --url https://token.actions.githubusercontent.com \
  --client-id-list sts.amazonaws.com \
  --thumbprint-list 6938fd4d98bab03faadb97b34396831e3780aea1
```

## Step 2: Create the IAM Role

Create a file called `trust-policy.json`:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Federated": "arn:aws:iam::YOUR_ACCOUNT_ID:oidc-provider/token.actions.githubusercontent.com"
      },
      "Action": "sts:AssumeRoleWithWebIdentity",
      "Condition": {
        "StringEquals": {
          "token.actions.githubusercontent.com:aud": "sts.amazonaws.com"
        },
        "StringLike": {
          "token.actions.githubusercontent.com:sub": "repo:YOUR_GITHUB_USERNAME/elgasos.serag.net:*"
        }
      }
    }
  ]
}
```

> **Important**: Replace `YOUR_ACCOUNT_ID` and `YOUR_GITHUB_USERNAME` with your actual values.

Create the role:

```bash
aws iam create-role \
  --role-name github-actions-deploy-portfolio \
  --assume-role-policy-document file://trust-policy.json
```

## Step 3: Create and Attach the Permissions Policy

Create a file called `permissions-policy.json`:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "S3BucketAccess",
      "Effect": "Allow",
      "Action": [
        "s3:PutObject",
        "s3:GetObject",
        "s3:DeleteObject",
        "s3:ListBucket"
      ],
      "Resource": [
        "arn:aws:s3:::YOUR_BUCKET_NAME",
        "arn:aws:s3:::YOUR_BUCKET_NAME/*"
      ]
    },
    {
      "Sid": "CloudFrontInvalidation",
      "Effect": "Allow",
      "Action": "cloudfront:CreateInvalidation",
      "Resource": "arn:aws:cloudfront::YOUR_ACCOUNT_ID:distribution/YOUR_DISTRIBUTION_ID"
    }
  ]
}
```

> **Note**: Remove the CloudFront statement if you're not using CloudFront.

Attach the policy:

```bash
aws iam put-role-policy \
  --role-name github-actions-deploy-portfolio \
  --policy-name s3-deploy-policy \
  --policy-document file://permissions-policy.json
```

## Step 4: Configure GitHub Repository Variables

Go to your GitHub repository → **Settings** → **Secrets and variables** → **Actions** → **Variables**

Add these **repository variables**:

| Variable Name | Value | Example |
|--------------|-------|---------|
| `AWS_REGION` | Your AWS region | `us-east-1` |
| `AWS_ROLE_ARN` | The role ARN from Step 2 | `arn:aws:iam::123456789012:role/github-actions-deploy-portfolio` |
| `S3_BUCKET` | Your S3 bucket name | `my-portfolio-bucket` |
| `CLOUDFRONT_DISTRIBUTION_ID` | (Optional) CloudFront distribution ID | `E1234567890ABC` |

## Step 5: Create the S3 Bucket (if not exists)

```bash
# Create bucket
aws s3 mb s3://YOUR_BUCKET_NAME --region YOUR_REGION

# Enable static website hosting
aws s3 website s3://YOUR_BUCKET_NAME \
  --index-document index.html \
  --error-document index.html

# Set bucket policy for public read (if not using CloudFront)
cat > bucket-policy.json << 'EOF'
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "PublicReadGetObject",
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::YOUR_BUCKET_NAME/*"
    }
  ]
}
EOF

aws s3api put-bucket-policy \
  --bucket YOUR_BUCKET_NAME \
  --policy file://bucket-policy.json
```

## Verification

After setup, push a commit to the `main` branch. The workflow should:

1. ✅ Assume the IAM role via OIDC
2. ✅ Build the application
3. ✅ Sync files to S3
4. ✅ Invalidate CloudFront cache (if configured)

## Security Notes

- **No long-lived credentials**: OIDC tokens are short-lived and scoped to specific workflows
- **Branch protection**: The trust policy can be restricted to specific branches (e.g., `ref:refs/heads/main`)
- **Minimal permissions**: The IAM policy grants only the required S3 and CloudFront permissions
