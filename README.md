# CloudFormation Nested Stack Template : Security Group.

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

---

This repository contains a reusable CloudFormation template, `cloudformation/template.yaml`, that creates an EC2 security group with up to five ingress rules. Each rule picks a named rule set and has its own source. The template is designed to be used as a nested stack from a parent stack, or deployed directly.

## Overview

The stack creates:

- One `AWS::EC2::SecurityGroup` in the VPC you supply
- Up to five `AWS::EC2::SecurityGroupIngress` rules, one per rule slot
- Default allow-all egress, as provided by EC2

Each ingress rule is created only when its rule set has a port defined for that slot, and its source is set. Rules and sources are independent, so you can mix CIDR and security group sources in one group.

## Repository Layout

```text
cloudformation/
├── template.yaml        # The security group stack template
├── parameters.json      # Parameter values used by CI
└── stack-config.json    # Stack name, template file, and parameter file used by CI

.github/workflows/       # CI, release, and automation workflows
.env/environments.yaml   # GitHub environments (ci, devl) and region (us-east-1)
```

## Parameters

| Parameter | Type | Default | Description |
| ----------- | ------ | --------- | ------------- |
| `VpcId` | `AWS::EC2::VPC::Id` | — | VPC in which to create the security group (required) |
| `ProjectName` | String | `proj-ztc` | Project name, the first part of the group name |
| `Environment` | String | — | Environment: `devl`, `stag`, or `prod` (required) |
| `SecurityGroupBaseName` | String | — | Base name, the middle part of the group name (required) |
| `CiSuffix` | String | `""` | Optional suffix appended to the group name, used for CI deployments |
| `SecurityGroupDescription` | String | `""` | Description. Leave blank to generate `Security group for <stack-name>` |
| `RuleNRuleSet` (N = 1–5) | String | — | Rule set for rule N. Choose `None` to skip the rule (required) |
| `RuleNSourceCidr` (N = 1–5) | String | `""` | IPv4 CIDR allowed to reach rule N. Set this or the security group ID, not both |
| `RuleNSourceSecurityGroupId` (N = 1–5) | String | `""` | Security group ID allowed to reach rule N. Set this or the CIDR, not both |

The console groups these parameters as VPC, name and description, then one group per rule.

## Rule Sets

Rule N is built from **slot N** of the rule set you choose for it. Each slot is `protocol, from port, to port`.

| Rule set | Rule 1 | Rule 2 | Rule 3 |
| ----------- | ------ | ------ | ------ |
| `None` | — | — | — |
| `WebServer` | tcp 80 | tcp 443 | — |
| `SSHOnly` | tcp 22 | — | — |
| `AppTier` | tcp 8000–8080 | tcp 9090 | — |
| `Dev` | tcp 22 | tcp 80 | tcp 8000–8080 |
| `RDSMySQL` | tcp 3306 | — | — |
| `AuroraMySQL` | tcp 3306 | — | — |
| `AuroraPostgreSQL` | tcp 5432 | — | — |
| `RDSPostgreSQL` | tcp 5432 | — | — |
| `RDSOracle` | tcp 1521 | — | — |
| `RDSSQLServer` | tcp 1433 | — | — |

Rules 4 and 5 are unused in every set. Each rule uses only its own slot, so `Rule1RuleSet=WebServer` opens port 80 and nothing else. To also open port 443, set `Rule2RuleSet=WebServer`.

The rule sets are defined in the `IngressRuleSets` mapping in `cloudformation/template.yaml`. To add a set, add an entry there and add its name to the `AllowedValues` of each `RuleNRuleSet` parameter.

## Naming

The group name is:

```bash
{ProjectName}-{SecurityGroupBaseName}-{Environment}-{Region}[-{CiSuffix}]
```

Example: `proj-ztc-web-devl-us-east-1`

The `Name` tag and the `SecurityGroupName` output use the same value.

Each ingress rule has a `Description` that shows its rule set, ports, and source, for example `WebServer rule 1: tcp 80-80 from 10.0.0.0/16`. Ingress rules do not support tags in CloudFormation, so the description is the way to label them.

## Outputs

| Output | Description |
| ------ | ----------- |
| `SecurityGroupId` | ID of the security group. Exported as `<stack-name>-SecurityGroupId` |
| `SecurityGroupName` | Name of the security group |

## Usage

### 1. Deploy with the AWS CLI

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
    Rule2RuleSet=None \
    Rule3RuleSet=None \
    Rule4RuleSet=None \
    Rule5RuleSet=None \
  --region us-east-1
```

Rules 2 through 5 are set to `None`, so only rule 1 is created.

### 2. Use as a nested stack

Upload the template to S3, then reference it from a parent stack:

```bash
aws s3 cp cloudformation/template.yaml s3://your-cfn-bucket/templates/security-group.yaml
```

```yaml
WebSecurityGroupStack:
  Type: AWS::CloudFormation::Stack
  Properties:
    TemplateURL: https://s3.amazonaws.com/your-cfn-bucket/templates/security-group.yaml
    Parameters:
      VpcId: !Ref VpcId
      ProjectName: myproject
      Environment: devl
      SecurityGroupBaseName: web
      Rule1RuleSet: WebServer
      Rule1SourceSecurityGroupId: !GetAtt AlbSecurityGroup.GroupId
      Rule2RuleSet: None
      Rule3RuleSet: None
      Rule4RuleSet: None
      Rule5RuleSet: None

Outputs:
  WebSecurityGroupId:
    Value: !GetAtt WebSecurityGroupStack.Outputs.SecurityGroupId
```

### Example: Web tier behind an Application Load Balancer

The ALB's security group accepts HTTP and HTTPS from the internet. The EC2 instances accept HTTP only from the ALB's security group.

```bash
# ALB security group
Rule1RuleSet=WebServer
Rule1SourceCidr=0.0.0.0/0
Rule2RuleSet=WebServer
Rule2SourceCidr=0.0.0.0/0

# EC2 web server security group
Rule1RuleSet=WebServer
Rule1SourceSecurityGroupId=sg-<ALB security group ID>
Rule2RuleSet=None
```

If the application listens on 8000–8080 or 9090, use `AppTier` instead of `WebServer` for the instances.

## Validation

```bash
aws cloudformation validate-template --template-body file://cloudformation/template.yaml
cfn-lint cloudformation/template.yaml
```

Both should pass with no errors or warnings.

## Limitations

- **No source check per slot.** If a rule has a rule set with a port but no source, the stack creates no ingress entry for that rule and CloudFormation reports no error. The `Rules` section cannot read the mapping, so this check is not possible there.
- **Only TCP is in the rule sets.** Protocols such as UDP or ICMP need new mapping entries.
- **Five rules at most.** Each stack creates at most five ingress entries.

## CI and Release

- **`ci.yaml`** runs on manual dispatch. It loads `cloudformation/stack-config.json` and calls the reusable CI build workflow with the `ci` environment.
- **`release.yaml`** runs on push to `main`. It runs semantic-release and creates the version tag and GitHub release.

Commits follow the conventional format:

- `feat:` → MINOR release
- `fix:` → PATCH release
- `BREAKING CHANGE:` → MAJOR release
- `chore:`, `docs:` → no release

Use `npx cz commit` to create a commit in this format.

## License

MIT
