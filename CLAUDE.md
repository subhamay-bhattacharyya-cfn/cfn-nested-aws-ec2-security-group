# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

This is a **CloudFormation template repository** that provides a template for creating an EC2 security group with up to five ingress rules. Each rule is chosen from a named rule set and has its own source, either a CIDR block or another security group.

**Key characteristics:**

- Single stack template: `cloudformation/template.yaml`
- Rule sets are defined in a `Mappings` section, so port choices live in one place
- Per-rule source parameters (CIDR or security group ID), with a check that only one is set
- Security group name built from project, base name, environment, region, and an optional CI suffix
- Automated semantic versioning and releases
- AWS OIDC authentication for CI/CD deployments

## Project Structure

```text
cloudformation/
├── template.yaml                  # EC2 security group template (the stack)
├── parameters.json                # Parameter values for the stack
└── stack-config.json              # Stack name, template file, and parameter file used by CI

.github/workflows/
├── ci.yaml                        # Validates and deploys the stack (manual trigger)
├── release.yaml                   # Semantic release on push to main
├── create-branch.yaml             # Auto-create feature branches from issues
├── claude.yaml                    # Claude Code workflow
├── claude-code-review.yaml        # Claude Code review workflow
├── notify.yaml                    # Notifications
└── setup-environments.yaml        # Creates the AWS environments (manual trigger)

.env/
└── environments.yaml              # GitHub environments (ci, devl) and region (us-east-1)

scripts/plugins/
├── release.config.js              # Semantic-release configuration
└── (other release plugins)        # Custom commit analysis, notes generation

.claude/
├── .skills/                       # Project skills (CI workflow, contribution, package.json, README)
├── settings.json                  # Claude Code workspace settings
└── settings.local.json            # Local overrides

.devcontainer/
└── devcontainer.json              # Dev container setup (Node.js 20)

package.json                       # Dependencies: semantic-release, commitizen
README.md                          # Template documentation and usage examples
```

## Development Commands

### Install dependencies

```bash
npm ci
```

### Trigger semantic release (usually automatic on main)

```bash
npm run release
```

### Commit with conventional commit format

```bash
npx cz commit
```

Select `feat`, `fix`, or `chore` type. Only `feat` and `fix` trigger releases.

## Key Architecture Concepts

### Rule Sets and Per-Rule Selection

Each rule N (1–5) has its own `RuleNRuleSet` parameter. The value is a key in the `IngressRuleSets` mapping, and rule N uses **slot N** of that set:

```yaml
IngressRuleSets:
  WebServer:
    Rule1: "tcp,80,80"
    Rule2: "tcp,443,443"
```

- `"protocol,fromPort,toPort"` is the slot format. `"none"` marks an unused slot.
- `None` is a rule set where every slot is `none`. Choose it to skip a rule.
- Choosing a set for rule N only picks up slot N. To get several ports from one set, set several rules to that set.

### Sources

- Each rule has `RuleNSourceCidr` and `RuleNSourceSecurityGroupId`. Set one or the other, not both.
- The `Rules` section asserts that not both are set.
- The rule is created only if its slot is not `none`. The ingress resource for the source type that is set is created.

### Security Group Naming

```bash
{ProjectName}-{SecurityGroupBaseName}-{Environment}-{Region}[-{CiSuffix}]
```

Example: `myproject-web-devl-us-east-1`

- `Environment` allows `devl`, `stag`, or `prod`.
- `CiSuffix` is optional and appended only when set.
- The `Name` tag and the `SecurityGroupName` output use the same value.

### Rule Descriptions

`AWS::EC2::SecurityGroupIngress` does not support tags, so each ingress rule has a `Description` instead, such as `WebServer rule 1: tcp 80-80 from 10.0.0.0/16`. Keep the `Description` in sync with the mapping.

## Key Files to Understand

### `cloudformation/template.yaml`

**Purpose:** Creates an EC2 security group and up to five ingress rules.

**Parameters:**

- `VpcId` (required): VPC for the security group
- `ProjectName` (required): First part of the group name
- `Environment` (required): `devl`, `stag`, or `prod`
- `SecurityGroupBaseName` (required): Middle part of the group name
- `CiSuffix` (optional): Suffix for CI deployments
- `SecurityGroupDescription` (optional): Generated from the stack name when blank
- `RuleNRuleSet`, `RuleNSourceCidr`, `RuleNSourceSecurityGroupId` for N = 1–5

**Outputs:**

- `SecurityGroupId`: Exported as `{StackName}-SecurityGroupId`
- `SecurityGroupName`: The generated group name

**Metadata:** `AWS::CloudFormation::Interface` groups the parameters in this order: VPC, name and description, then each rule's rule set and sources.

### `cloudformation/parameters.json`

Parameter values used by CI. Keep its keys in line with `template.yaml`.

### `cloudformation/stack-config.json`

```json
{
  "stackName": "...",
  "templateFile": "cloudformation/template.yaml",
  "parameterFile": "cloudformation/parameters.json"
}
```

### `.github/workflows/ci.yaml`

**Triggered on:** Manual `workflow_dispatch` only. The push and pull request triggers are commented out.

**Phases:**

1. **Load config:** reads `cloudformation/stack-config.json`
2. **CI build:** calls the reusable workflow `subhamay-bhattacharyya-gha/cfn-ci-build-reusable-wf` with the `ci` environment, the stack name, the template, and the parameter file
3. **Changelog:** on `main`, generates the changelog

**Environment setup:**

- Uses AWS OIDC for keyless authentication
- Requires GitHub environment `ci` with OIDC trust configured

### `.github/workflows/release.yaml`

**Triggered:** On push to main

**Release rules:**

- `feat:` → MINOR bump (0.1.0 → 0.2.0)
- `fix:` → PATCH bump (0.1.0 → 0.1.1)
- `BREAKING CHANGE:` → MAJOR bump (0.1.0 → 1.0.0)
- Other commits → no release

## Testing & Validation

```bash
aws cloudformation validate-template --template-body file://cloudformation/template.yaml
cfn-lint cloudformation/template.yaml
```

**Manual stack deployment:**

```bash
aws cloudformation deploy \
  --template-file cloudformation/template.yaml \
  --stack-name my-sg-devl \
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
    Rule5RuleSet=None \
  --region us-east-1
```

Run `cfn-lint` before a PR. It should report no errors or warnings.

## AWS Credentials & Environment Variables

**GitHub environments:** `ci` and `devl`, both mapped to `AWS-CFN-TEMPLATES` in `.env/environments.yaml`. Region: `us-east-1`.

**OIDC setup:** The CI workflow uses AWS OIDC for keyless auth. The GitHub OIDC provider must trust the role used by the environment.

## Conventional Commits & Release Flow

This repo enforces conventional commits to drive semantic versioning:

```bash
npx cz commit
```

Commit types:

- `feat: add support for X` → triggers MINOR release
- `fix: correct behavior of Y` → triggers PATCH release
- `chore: update deps` → no release
- `docs: clarify README` → no release

Only commits to `main` trigger releases. Feature branches use this format but releases happen on merge to main.

## When Modifying the Template

1. **Edit `cloudformation/template.yaml`**
2. **Update the mapping** if a rule set changes, and keep each rule's `Description` in sync
3. **Update `cloudformation/parameters.json`** if parameters are added, renamed, or removed
4. **Update the `Metadata` parameter groups** so new parameters appear in the console
5. **Validate** with `aws cloudformation validate-template` and `cfn-lint`
6. **Create a PR** with a conventional commit message (e.g., `feat: add rule set for Redis`)
7. **Merge to main** → release workflow creates the version tag and GitHub release

## Dev Container

Pre-configured with:

- Node.js 20
- GitHub Copilot extension

Use via VS Code: `code --remote-container-url <repo-url>`

## Current Branch

Main branch is the release branch. Feature work branches from here and merges back via PR. Branch naming follows: `{type}/CFN-{issue-number}-{slug}` (e.g., `feature/CFN-42-add-encryption`).
