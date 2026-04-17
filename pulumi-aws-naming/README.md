# Pulumi AWS Naming

Consistent naming conventions for Pulumi-managed AWS resources.

## Install

```
npx skills add sudorock/agent-skills --skill pulumi-aws-naming
```

## What It Does

Defines a naming system for both Pulumi logical names (URN/state identity) and AWS physical names (console/logs/metrics identity), built from five components: namespace, service, env, resource-type, and parts.

**Logical name** (Pulumi constructor arg):
```
{namespace}-{service}-{resource-type}-{parts}
```

**Physical name** (AWS console):
```
{namespace}-{service}-{env}-{resource-type}-{parts}
```

### Example

```
Logical:   acme-payments-lambda-handle-webhook
Physical:  acme-payments-prod-lambda-handle-webhook
```

With formatting exceptions for IAM (PascalCase), CloudWatch/SSM (slash-separated), and S3 (globally unique).

### Included

- Naming rules and formatting constraints
- Logical-to-physical name mapping with examples
- Resource type abbreviation table covering 90+ AWS resource types
- Handling of IAM, CloudWatch, SSM, and S3 naming exceptions

## System Requirements

None. This is a pure naming convention — no external tools needed.
