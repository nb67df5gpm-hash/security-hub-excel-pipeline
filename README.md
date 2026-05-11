# AWS Security Hub → Audit-Ready Excel Pipeline

> A hands-on cloud-security lab where I built a serverless AWS pipeline that pulls findings from **AWS Security Hub** and turns them into a professional, multi-sheet Excel report ready for auditors.

**Built by:** [Shuayb](https://www.linkedin.com/in/shu-/) — student / self-learner exploring cloud security, AWS, and Python automation.

---

## What this project does

This pipeline answers a real GRC problem: *security teams produce findings in dashboards, but auditors want Excel spreadsheets.* Instead of manually exporting and reformatting, this Lambda function pulls every finding from Security Hub and writes a structured `.xlsx` to S3 — on demand.

**Live result from my run:** 466 real Security Hub findings exported into 3 worksheets (`Executive Summary`, `Detailed Findings`, `Pivot Analysis`).

## Architecture

```
┌─────────────────┐    ┌──────────────────┐    ┌─────────────────┐
│   Security Hub  │───▶│  Lambda (Py 3.9) │───▶│   S3 Bucket     │
│   Findings API  │    │  boto3 + openpyxl│    │  Excel Reports  │
└─────────────────┘    └──────────────────┘    └─────────────────┘
```

**AWS services used:** Security Hub · Lambda · S3 · IAM · CloudFormation

## Tech stack

- **Python 3.9** — Lambda runtime
- **boto3** — AWS SDK (Security Hub `get_findings` pagination)
- **openpyxl** — Excel generation with conditional formatting, headers, and charts
- **CloudFormation** — Infrastructure as code (Lambda, IAM role, log group)
- **AWS CLI v2** — Deployment + invocation

## What I learned

This was a learning project, so here's the honest list of things that bit me and how I worked through them:

- **IAM least-privilege is real.** My lab IAM user couldn't `CreateRole`, then couldn't `DeleteLogGroup` on rollback. I learned to read CloudFormation `describe-stack-events` to find the actual blocking permission.
- **AWS managed-policy quota.** Users are capped at 10 attached managed policies. I hit the limit and learned that **inline policies don't count toward the quota** — a useful workaround for lab environments.
- **`CAPABILITY_NAMED_IAM`** is required when a CFN template creates IAM resources with explicit names. It's a deliberate guard rail, not a bug.
- **Lambda packaging.** Dependencies (`openpyxl`) have to be vendored into the deployment zip alongside `lambda_function.py` — Lambda runtimes don't `pip install` for you.
- **Failed stacks can get stuck** in `DELETE_FAILED` when the user lacks cleanup permissions. Fixing the permission, then re-running `delete-stack`, was the path out.

## Deployment

> **Prerequisites:** AWS CLI configured, Security Hub enabled in the account, IAM permissions for Lambda + S3 + IAM + CloudFormation.

### 1. Package the Lambda

```bash
pip install -r requirements.txt -t lib/
cd lib && zip -r ../lambda-source.zip . && cd ..
zip -g lambda-source.zip lambda_function.py
```

### 2. Upload source to S3

```bash
export AWS_PROFILE=your-profile
export BUCKET_NAME="security-hub-reports-$(date +%s)"

aws s3 mb s3://$BUCKET_NAME
aws s3 cp lambda-source.zip s3://$BUCKET_NAME/source/lambda-source.zip
```

### 3. Deploy with CloudFormation

```bash
aws cloudformation deploy \
  --template-file cloudformation-template.yaml \
  --stack-name security-hub-excel-pipeline \
  --capabilities CAPABILITY_NAMED_IAM \
  --parameter-overrides S3BucketName=$BUCKET_NAME
```

### 4. Generate a report

```bash
aws lambda invoke \
  --function-name security-hub-excel-generator-cf \
  --output json response.json

cat response.json
aws s3 ls s3://$BUCKET_NAME/reports/
```

### 5. Download

```bash
aws s3 cp s3://$BUCKET_NAME/reports/security_hub_report_YYYYMMDD_HHMMSS.xlsx ./report.xlsx
open report.xlsx
```

## What's in the Excel report

- **Executive Summary** — total findings, severity breakdown, top compliance frameworks
- **Detailed Findings** — every finding with title, severity, resource, remediation URL
- **Pivot Analysis** — findings grouped by severity / resource type for quick filtering

## Cleanup

```bash
aws cloudformation delete-stack --stack-name security-hub-excel-pipeline
aws s3 rm s3://$BUCKET_NAME --recursive
aws s3 rb s3://$BUCKET_NAME
```

## Troubleshooting (from my own pain)

| Symptom | Likely cause | Fix |
|---|---|---|
| `CREATE_FAILED` on `LambdaExecutionRole` | User lacks `iam:CreateRole` | Attach admin perms (or scoped IAM policy) to your deploying user |
| `Cannot attach more managed policies` | Hit the 10-policy quota | Use an **inline** policy or a group |
| `DELETE_FAILED` on `FunctionLogGroup` | User lacks `logs:DeleteLogGroup` | Add the permission, then re-run `delete-stack` |
| Lambda runs but Excel has 0 findings | Security Hub not enabled / no findings | Enable Security Hub + standards in the region |

## Credit

Built from a tutorial scaffold and extended/documented as a learning exercise. Repo and notes maintained by [Shuayb](https://www.linkedin.com/in/shu-/).
