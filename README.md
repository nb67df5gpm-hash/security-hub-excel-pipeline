
# AWS Security Hub to Excel Pipeline

*Automated security findings extraction with audit-ready Excel reporting*

## Overview

This project demonstrates how to build a serverless pipeline that extracts security findings from AWS Security Hub and generates professional Excel reports. It bridges the gap between GRC engineering automation and audit requirements by delivering data in the format compliance teams actually use.

## Architecture

```
┌─────────────────┐    ┌──────────────────┐    ┌─────────────────┐
│   Security Hub  │───▶│  Lambda Function │───▶│   S3 Bucket     │
│   Findings      │    │  (Python 3.9)   │    │  (Excel Files)  │
└─────────────────┘    └──────────────────┘    └─────────────────┘
```

**AWS Services Used:**
- **AWS Security Hub** - Centralized security findings aggregation
- **AWS Lambda** - Serverless compute for data processing
- **Amazon S3** - Storage for generated Excel reports
- **AWS IAM** - Role-based access control
- **CloudFormation** - Infrastructure as code deployment

## Key Features

- **Multi-worksheet Excel reports** with executive summaries, detailed findings, and pivot analysis
- **Professional formatting** with conditional formatting and charts
- **Serverless architecture** for cost-effective, scalable execution
- **Infrastructure as code** deployment with CloudFormation
- **Simplified dependencies** using only boto3 and openpyxl

## Quick Deployment

**Prerequisites:**
- AWS CLI installed and configured
- AWS Security Hub enabled in your account
- IAM permissions for Lambda, S3, Security Hub, and CloudFormation

**Step 1: Set up your environment**
```bash
# Set your AWS profile (replace 'your-profile' with your actual profile name)
export AWS_PROFILE=your-profile

# Verify AWS access
aws sts get-caller-identity
```

**Step 2: Create S3 bucket and upload source**
```bash
# Create unique S3 bucket
export BUCKET_NAME="security-hub-reports-$(date +%s)"
echo "Creating bucket: $BUCKET_NAME"
exitaws s3 mb s3://$BUCKET_NAME

# Upload the provided source code package
aws s3 cp lambda-source.zip s3://$BUCKET_NAME/source/lambda-source.zip
```

**Step 3: Deploy infrastructure**
```bash
# Deploy CloudFormation stack
aws cloudformation deploy \
  --template-file cloudformation-template.yaml \
  --stack-name security-hub-excel-pipeline \
  --capabilities CAPABILITY_NAMED_IAM \
  --parameter-overrides S3BucketName=$BUCKET_NAME

echo "Deployment complete!"
```

**Step 4: Test the function**
```bash
# Generate your first Excel report
aws lambda invoke \
  --function-name security-hub-excel-generator-cf \
  --output json \
  response.json

# Check the response
cat response.json

# View generated reports in S3
aws s3 ls s3://$BUCKET_NAME/reports/
```

**Step 5: Download your report**
```bash
# List available reports
aws s3 ls s3://$BUCKET_NAME/reports/

# Download the latest report (replace filename with actual file)
aws s3 cp s3://$BUCKET_NAME/reports/security_hub_report_YYYYMMDD_HHMMSS.xlsx ./my-security-report.xlsx

# Open the Excel file
open my-security-report.xlsx  # macOS
# or
start my-security-report.xlsx  # Windows
```

## What You'll Get

Your Excel reports will include:

- **Executive Summary** - High-level metrics and severity breakdown
- **Detailed Findings** - Complete finding data with remediation links  
- **Pivot Analysis** - Interactive tables for deeper analysis
- **Professional formatting** - Conditional formatting, headers, and charts

## Cleanup (Optional)

To remove all resources when you're done:

```bash
# Delete the CloudFormation stack
aws cloudformation delete-stack --stack-name security-hub-excel-pipeline

# Remove S3 bucket and contents
aws s3 rm s3://$BUCKET_NAME --recursive
aws s3 rb s3://$BUCKET_NAME
```

## Troubleshooting

**Common Issues:**
- **Permission errors**: Ensure your AWS profile has Security Hub, Lambda, S3, and CloudFormation permissions
- **No findings returned**: Verify Security Hub is enabled and has findings in your account
- **Deployment failures**: Check CloudFormation stack events for detailed error messages

**Support:**
This solution has been tested with 1000+ real Security Hub findings and generates professional audit-ready Excel reports.
