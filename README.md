# CloudFormation Nested Stack -  EC2 Security Group

<!-- Row 1: Status - Most Important -->
[![Release](https://img.shields.io/github/v/release/subhamay-bhattacharyya-cfn/cfn-nested-aws-ec2-security-group?label=Release)](https://github.com/subhamay-bhattacharyya-cfn/cfn-nested-aws-ec2-security-group/releases)&nbsp;[![Release Workflow](https://github.com/subhamay-bhattacharyya-cfn/cfn-nested-aws-ec2-security-group/actions/workflows/release.yaml/badge.svg)](https://github.com/subhamay-bhattacharyya-cfn/cfn-nested-aws-ec2-security-group/actions/workflows/release.yaml)&nbsp;[![Issues](https://img.shields.io/github/issues/subhamay-bhattacharyya-cfn/cfn-nested-aws-ec2-security-group)](https://github.com/subhamay-bhattacharyya-cfn/cfn-nested-aws-ec2-security-group/issues)&nbsp;[![Last Commit](https://img.shields.io/github/last-commit/subhamay-bhattacharyya-cfn/cfn-nested-aws-ec2-security-group)](https://github.com/subhamay-bhattacharyya-cfn/cfn-nested-aws-ec2-security-group/commits)

<!-- Row 2: Code Quality -->
[![Top Language](https://img.shields.io/badge/Languages-Python%20%7C%20YAML-blue)](https://github.com/subhamay-bhattacharyya-cfn/cfn-nested-aws-ec2-security-group)&nbsp;[![Commits](https://img.shields.io/github/commit-activity/t/subhamay-bhattacharyya-cfn/cfn-nested-aws-ec2-security-group)](https://github.com/subhamay-bhattacharyya-cfn/cfn-nested-aws-ec2-security-group/commits)

<!-- Row 3: Tech Stack -->
[![CloudFormation](https://img.shields.io/badge/CloudFormation-IaC-orange?logo=amazonaws&logoColor=white)](https://aws.amazon.com/cloudformation/)&nbsp;[![Built with Claude Code](https://img.shields.io/badge/Built_with-Claude_Code-D97757?logo=anthropic&logoColor=white)](https://claude.ai/)

<!-- Row 4: Repository Info -->
[![Files](https://img.shields.io/github/directory-file-count/subhamay-bhattacharyya-cfn/cfn-nested-aws-ec2-security-group)](https://github.com/subhamay-bhattacharyya-cfn/cfn-nested-aws-ec2-security-group)&nbsp;[![Repo Size](https://img.shields.io/github/repo-size/subhamay-bhattacharyya-cfn/cfn-nested-aws-ec2-security-group)](https://github.com/subhamay-bhattacharyya-cfn/cfn-nested-aws-ec2-security-group)&nbsp;[![Release Date](https://img.shields.io/github/release-date/subhamay-bhattacharyya-cfn/cfn-nested-aws-ec2-security-group)](https://github.com/subhamay-bhattacharyya-cfn/cfn-nested-aws-ec2-security-group/releases)

<!-- Row 5: Custom Metrics -->
[![Custom Endpoint](https://img.shields.io/endpoint?url=https://gist.githubusercontent.com/bsubhamay/a512571f1782749bed95bfff876baded/raw/cfn-nested-aws-ec2-security-group.json)](https://gist.github.com/subhamay-bhattacharyya/a512571f1782749bed95bfff876baded)

## EC2 Security Group Template (cloudformation/template.yaml)

Creates an EC2 security group with up to five ingress rules. Each rule selects a named rule set and has its own source, either a CIDR block or another security group. Egress uses the default allow-all rule.

### Parameters

| Parameter | Type | Default | Description |
| ----------- | ------ | --------- | ------------- |
| `VpcId` | `AWS::EC2::VPC::Id` | — | VPC in which to create the security group |
| `ProjectName` | String | — | Project name, the first part of the group name |
| `Environment` | String | — | Environment: `devl`, `stag`, or `prod` |
| `SecurityGroupBaseName` | String | — | Base name, the middle part of the group name |
| `CiSuffix` | String | `""` | Optional suffix appended to the group name (CI deployments) |
| `SecurityGroupDescription` | String | `""` | Description. Leave blank to generate one from the stack name and rule set |
| `RuleNRuleSet` (N = 1–5) | String | — | Rule set for rule N, or `None` to skip the rule |
| `RuleNSourceCidr` (N = 1–5) | String | `""` | CIDR block allowed to reach rule N (set this or the security group ID, not both) |
| `RuleNSourceSecurityGroupId` (N = 1–5) | String | `""` | Security group ID allowed to reach rule N (set this or the CIDR, not both) |

### Rule Sets

Rule N is built from slot N of the rule set chosen for it.

| Rule set | Rule 1 | Rule 2 | Rule 3 |
| ----------- | ------ | ------ | ------ |
| `WebServer` | tcp 80 | tcp 443 | — |
| `SSHOnly` | tcp 22 | — | — |
| `AppTier` | tcp 8000–8080 | tcp 9090 | — |
| `Dev` | tcp 22 | tcp 80 | tcp 8000–8080 |
| `RDSMySQL`, `AuroraMySQL` | tcp 3306 | — | — |
| `AuroraPostgreSQL`, `RDSPostgreSQL` | tcp 5432 | — | — |
| `RDSOracle` | tcp 1521 | — | — |
| `RDSSQLServer` | tcp 1433 | — | — |

Rules 4 and 5 are empty in every set.

### Naming

The security group is named:

```bash
{ProjectName}-{SecurityGroupBaseName}-{Environment}-{Region}
```

If `CiSuffix` is set, `-{CiSuffix}` is appended. The `Name` tag uses the same value.

Each ingress rule has a `Description` such as `WebServer rule 1: tcp 80-80 from 10.0.0.0/16`, because ingress rules cannot carry tags in CloudFormation.

### Deploy

```bash
aws cloudformation deploy \
  --template-file cloudformation/template.yaml \
  --stack-name web-sg-devl \
  --parameter-overrides \
    VpcId=vpc-0123456789abcdef0 \
    ProjectName=myproject \
    Environment=devl \
    SecurityGroupBaseName=web \
    Rule1RuleSet=WebServer \
    Rule1SourceCidr=10.0.0.0/16 \
    Rule2RuleSet=WebServer \
    Rule2SourceCidr=10.0.0.0/16 \
    Rule3RuleSet=None \
    Rule4RuleSet=None \
    Rule5RuleSet=None
```

Rule 2 must be set to `WebServer` as well to get the HTTPS port, because each rule reads its own slot.


---

This repository contains nested CloudFormation templates for deploying S3 buckets with security best practices and optional policy enforcement.

## Overview

This is a **nested stack template** designed to be invoked from a parent/root CloudFormation stack. Templates are stored in this repository and should be uploaded to an S3 bucket for reference by parent stacks.

## Template Files

### CloudFormation Templates

- **`templates/s3-bucket.yaml`** — Nested template for S3 bucket creation (versioning, public access blocking)
- **`templates/s3-bucket-policy.yaml`** — Optional nested template for S3 bucket policy (encryption enforcement, secure transport)

### Parameter Files

- **`parameters/parameters.json`** — Parameter values for development environment

## Template Features

### S3 Bucket Template (s3-bucket.yaml)

- ✅ Versioning enabled by default
- ✅ Public Access Blocking
- ✅ Smart bucket naming (project prefix, account ID, environment, region)
- ✅ Optional CI suffix support

### S3 Bucket Policy Template (s3-bucket-policy.yaml)

- ✅ Encryption enforcement on uploads
- ✅ Secure transport enforcement (HTTPS only)
- ✅ Optional/conditional policy rules

## Parameters

### S3 Bucket Parameters

| Parameter | Type | Default | Description |
| ----------- | ------ | --------- | ------------- |
| `ProjectName` | String | — | Project name to use as bucket prefix (required) |
| `BucketBaseName` | String | `cfn-bucket` | Base name for S3 bucket |
| `environment` | String | `devl` | Deployment environment (devl, stag, prod) |
| `CiSuffix` | String | `""` | Optional CI suffix to append to bucket name |

### S3 Bucket Policy Parameters

**Generic (Standalone) Mode:**

| Parameter | Type | Description |
| ----------- | ------ | ------------- |
| `BucketName` | String | Direct bucket name (use for any bucket) |

**Integrated Mode (with bucket template):**

| Parameter | Type | Default | Description |
| ----------- | ------ | --------- | ------------- |
| `ProjectName` | String | — | Project name (must match bucket template) |
| `BucketBaseName` | String | `cfn-bucket` | Base name (must match bucket template) |
| `environment` | String | `devl` | Environment (must match bucket template) |
| `CiSuffix` | String | `""` | CI suffix (must match bucket template) |

**Usage:** If `BucketName` is provided (non-empty), it takes precedence. Otherwise, the bucket name is constructed from ProjectName/BucketBaseName/environment/CiSuffix.

## Outputs

### S3 Bucket Template Outputs

- `S3BucketName` — S3 bucket name
- `S3BucketArn` — S3 bucket ARN

### S3 Bucket Policy Template Outputs

- `BucketName` — Bucket name with policy applied
- `PolicyStatus` — Policy application status (Applied)

## Usage

### 1. Upload Templates to S3

```bash
aws s3 cp templates/s3-bucket.yaml s3://your-cfn-bucket/templates/s3-bucket.yaml
aws s3 cp templates/s3-bucket-policy.yaml s3://your-cfn-bucket/templates/s3-bucket-policy.yaml
```

### 2. Reference from Parent Stack

In your parent/root CloudFormation template:

```yaml
S3BucketNestedStack:
  Type: AWS::CloudFormation::Stack
  Properties:
    TemplateURL: https://s3.amazonaws.com/your-cfn-bucket/templates/s3-bucket.yaml
    Parameters:
      ProjectName: !Ref ProjectName
      BucketBaseName: cfn-bucket
      environment: !Ref Environment
      CiSuffix: !Ref CiSuffix
    Tags:
      - Key: Environment
        Value: !Ref Environment

S3PolicyNestedStack:
  Type: AWS::CloudFormation::Stack
  DependsOn: S3BucketNestedStack
  Properties:
    TemplateURL: https://s3.amazonaws.com/your-cfn-bucket/templates/s3-bucket-policy.yaml
    Parameters:
      BucketName: !GetAtt S3BucketNestedStack.Outputs.S3BucketName
      ProjectName: ""
      BucketBaseName: cfn-bucket
      environment: !Ref Environment
      CiSuffix: !Ref CiSuffix
    Tags:
      - Key: Environment
        Value: !Ref Environment

Outputs:
  BucketName:
    Value: !GetAtt S3BucketNestedStack.Outputs.S3BucketName
  BucketArn:
    Value: !GetAtt S3BucketNestedStack.Outputs.S3BucketArn
```

### 3. Deploy Using AWS CLI

#### Option A: Deploy Bucket Only

```bash
# Development (without CI prefix)
aws cloudformation create-stack \
  --stack-name cfn-s3-bucket-dev \
  --template-body file://templates/s3-bucket.yaml \
  --parameters file://parameters/dev.json

# Staging (without CI prefix)
aws cloudformation create-stack \
  --stack-name cfn-s3-bucket-stag \
  --template-body file://templates/s3-bucket.yaml \
  --parameters file://parameters/staging.json

# Production (without CI prefix)
aws cloudformation create-stack \
  --stack-name cfn-s3-bucket-prod \
  --template-body file://templates/s3-bucket.yaml \
  --parameters file://parameters/prod.json
```

#### Option B: Deploy Bucket + Policy (Recommended)

```bash
# Deploy bucket first
aws cloudformation create-stack \
  --stack-name cfn-s3-bucket-dev \
  --template-body file://templates/s3-bucket.yaml \
  --parameters file://parameters/dev.json

# Wait for bucket to be created
aws cloudformation wait stack-create-complete --stack-name cfn-s3-bucket-dev

# Get the bucket name from stack outputs
BUCKET_NAME=$(aws cloudformation describe-stacks \
  --stack-name cfn-s3-bucket-dev \
  --query 'Stacks[0].Outputs[?OutputKey==`S3BucketName`].OutputValue' \
  --output text)

#### Using Integrated Mode (with bucket template parameters)

```bash
aws cloudformation create-stack \
  --stack-name cfn-s3-policy-dev \
  --template-body file://templates/s3-bucket-policy.yaml \
  --parameters file://parameters/policy-dev.json
```

#### Using Generic Mode (standalone with direct bucket name)

```bash
aws cloudformation create-stack \
  --stack-name cfn-s3-policy-dev \
  --template-body file://templates/s3-bucket-policy.yaml \
  --parameters \
    ParameterKey=BucketName,ParameterValue=my-existing-bucket \
    ParameterKey=ProjectName,ParameterValue=""
```

#### Option C: Deploy with CI Suffix

```bash
# Development with CI suffix (e.g., for GitLab CI)
aws cloudformation create-stack \
  --stack-name cfn-s3-bucket-dev-ci \
  --template-body file://templates/s3-bucket.yaml \
  --parameters \
    ParameterKey=ProjectName,ParameterValue=myproject \
    ParameterKey=BucketBaseName,ParameterValue=cfn-bucket \
    ParameterKey=environment,ParameterValue=devl \
    ParameterKey=CiSuffix,ParameterValue=$CI_PIPELINE_ID
```

## Bucket Naming Convention

The templates generate bucket names using the following pattern:

**Without CI Suffix:**

```bash
{ProjectName}-{BucketBaseName}-{AccountId}-{Environment}-{Region}
```

Example: `myproject-cfn-bucket-123456789012-devl-us-east-1`

**With CI Suffix:**

```bash
{ProjectName}-{BucketBaseName}-{AccountId}-{Environment}-{Region}-{CiSuffix}
```

Example: `myproject-cfn-bucket-123456789012-devl-us-east-1-pipeline-12345`

## Best Practices Implemented

- ✅ Versioning enabled by default
- ✅ Public access blocked by default
- ✅ Smart bucket naming with project prefix, account ID, environment, and region
- ✅ Optional CI suffix support for unique deployments
- ✅ Optional policy enforcement (encryption and secure transport)
- ✅ Export values for cross-stack references

## License

MIT
