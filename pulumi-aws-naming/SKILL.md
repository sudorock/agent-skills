---
name: pulumi-aws-naming
description: Naming conventions for Pulumi-managed AWS resources. Use when writing Pulumi code that creates AWS resources, asking about naming conventions, or reviewing resource organization. Apply automatically when generating Pulumi AWS infrastructure code.
---

# AWS Resource Naming & Tagging

Covers two concerns for Pulumi-managed AWS infrastructure: Pulumi logical naming (URN/state identity) and AWS physical naming (console/logs/metrics identity).

## Rules

### Name Components

Every name is built from five components. If the user hasn't specified namespace or service, ask.

- **`namespace`** -- Stack config or constant. 2-8 lowercase chars, no hyphens. e.g. `acme`, `zephyr`, `mkt`
- **`service`** -- Stack config. Lowercase, hyphens OK, <=15 chars. e.g. `payments`, `platform`, `auth`
- **`env`** -- Pulumi stack name (`pulumi.getStack()`). Fixed set: `dev`, `stg`, `prod`
- **`resource-type`** -- Resource Type Abbreviations table. Bare service abbreviation for single-resource services; `{service}{type}` compound for multi-resource services. e.g. `vpc`, `sg`, `lambda`, `iamrole`, `cwlog`
- **`parts`** -- Context-dependent; each segment narrows scope. Zero or more segments ordered broad-to-specific. e.g. `api > ingress`, `webhook > dlq`

When multiple resources share a type, at least one part is required. Segments are joined by hyphens (default), slashes (CloudWatch/SSM), or concatenated in PascalCase (IAM).

### Formatting

- Lowercase and hyphens only (exceptions: IAM uses PascalCase, CloudWatch/SSM use slashes)
- Total length under 63 characters (fits most restrictive AWS limits)
- Start and end with alphanumeric, never a hyphen
- No consecutive hyphens (`--`)
- Environment always explicit, even when accounts separate environments

## Pulumi Logical Naming

```text
{namespace}-{service}-{resource-type}-{parts}
```

The first constructor argument. Includes namespace and service for identity within Pulumi state; env is excluded because the stack provides it. Always lowercase-hyphens — formatting exceptions (IAM PascalCase, CloudWatch/SSM slashes) apply only to physical names.

- Lowercase and hyphens only
- Under 63 characters
- Unique within the Pulumi program
- Describes role, not environment

## AWS Physical Naming

```text
{namespace}-{service}-{env}-{resource-type}-{parts}
```

Fully qualified. Appears in the AWS console, CloudWatch metrics, cost explorer, VPC flow logs, and cross-account contexts. Always set an explicit physical name to ensure human-readable identifiers across all surfaces.

### Exceptions

**IAM (PascalCase)** -- `{Namespace}{Service}{Env}{Parts}{ResourceType}`

**CloudWatch / SSM (slash-separated)** -- `/{namespace}/{service}/{env}/{parts}`

**S3 (globally unique)** -- `{namespace}-{service}-{env}-{parts}`

**Physical name property** -- The property that carries the physical name varies by resource: `name` (most resources), `identifier` (RDS), `bucket` (S3), `tags.Name` (VPC, Subnet, Internet Gateway, NAT Gateway, Route Table, Elastic IP). Use whichever property the resource exposes to set the standard physical name.

### Examples

**Parts decomposition** -- how resource-type and parts compose:

```text
sg-api-ingress         resource-type: sg,     parts: api > ingress      (target > direction)
subnet-private-a       resource-type: subnet, parts: private > a        (scope > AZ)
sqs-webhook-dlq        resource-type: sqs,    parts: webhook > dlq      (source > variant)
lambda-handle-webhook  resource-type: lambda, parts: handle > webhook   (action > source)
```

**Logical to physical name mapping** (env = `prod`):

```text
Logical Name                              Physical Name
──────────────────────────────────        ────────────────────────────────────────────────
acme-payments-vpc-main                    acme-payments-prod-vpc-main
acme-payments-subnet-private-a            acme-payments-prod-subnet-private-a
acme-payments-sg-api-ingress              acme-payments-prod-sg-api-ingress
acme-payments-alb-public                  acme-payments-prod-alb-public
acme-payments-lambda-handle-webhook       acme-payments-prod-lambda-handle-webhook
acme-payments-rds-primary                 acme-payments-prod-rds-primary
acme-payments-sqs-webhook-dlq             acme-payments-prod-sqs-webhook-dlq
acme-payments-iamrole-lambda-exec         AcmePaymentsProdLambdaExecRole              (IAM)
acme-payments-iampolicy-s3-read           AcmePaymentsProdS3ReadPolicy                (IAM)
acme-payments-cwlog-api-access            /acme/payments/prod/api/access              (CloudWatch)
acme-payments-ssmpar-db-connection        /acme/payments/prod/db/connection            (SSM)
acme-payments-s3-artifacts                acme-payments-prod-artifacts                (S3)
acme-payments-s3-access-logs              acme-payments-prod-access-logs              (S3)
```

## Resource Type Abbreviations

Use consistently in both logical and physical names. When a resource type isn't listed: if the AWS service has one commonly provisioned resource type, use the bare service abbreviation. Otherwise, concatenate the service abbreviation with a short type suffix (e.g., `cwalarm`, `cwdash`).

See `references/aws-resource-type-abbreviations.md` for the full list.
