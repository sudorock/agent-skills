# AWS Resource Type Abbreviations

Compound `{service}{type}` rule: bare service abbreviation for single-resource services, compound for multi-resource services.

## Networking

- VPC: `vpc`
- Subnet: `subnet`
- Internet Gateway: `igw`
- NAT Gateway: `natgw`
- EC2 Route: `route`
- Route Table: `rt`
- Route Table Association: `rta`
- Security Group: `sg`
- Network ACL: `nacl`
- Elastic IP: `eip`
- VPC Endpoint: `vpce`
- VPC Peering Connection: `vpcpeer`
- Network Interface: `eni`

## Load Balancing

- Application Load Balancer: `alb`
- Network Load Balancer: `nlb`
- Target Group: `tg`
- LB Listener: `lbl`
- LB Listener Rule: `lblrule`

## Compute

- EC2 Instance: `ec2`
- AMI: `ami`
- EBS Volume: `ebs`
- Key Pair: `keypair`
- Auto Scaling Group: `asg`
- Launch Template: `lt`

## ECS

- ECS Cluster: `ecscluster`
- ECS Service: `ecssvc`
- ECS Task Definition: `ecstd`
- ECS Capacity Provider: `ecscp`

## EKS

- EKS Cluster: `ekscluster`
- EKS Node Group: `eksng`
- EKS Addon: `eksaddon`
- EKS Fargate Profile: `eksfg`

## Container Registry

- ECR Repository: `ecr`

## IAM

- IAM Role: `iamrole`
- IAM Policy: `iampolicy`
- IAM Role Policy Attachment: `iampolicyattach`
- IAM User: `iamuser`
- IAM Group: `iamgroup`
- IAM Instance Profile: `iamip`
- IAM OIDC Provider: `iamoidc`

## Lambda

- Lambda Function: `lambda`
- Lambda Layer: `lambdalayer`
- Lambda Event Source Mapping: `lambdaesm`

## API Gateway

- API Gateway REST API: `apigw`
- API Gateway v2 (HTTP): `apigwv2`

## Step Functions

- SFN State Machine: `sfn`

## EventBridge

- EventBridge Rule: `ebrule`
- EventBridge Bus: `ebbus`

## S3

- S3 Bucket: `s3`
- S3 Bucket Policy: `s3policy`
- S3 Bucket Public Access Block: `s3pab`
- S3 Bucket Versioning V2: `s3versioning`

## RDS

- RDS Instance: `rds`
- RDS Aurora Cluster: `rdscluster`
- RDS Subnet Group: `rdssubgrp`
- RDS Parameter Group: `rdspg`
- RDS Proxy: `rdsproxy`

## DynamoDB

- DynamoDB Table: `ddb`

## ElastiCache

- ElastiCache Cluster: `redis` / `memcached`
- ElastiCache Replication Group: `redisrg`
- ElastiCache Subnet Group: `redissubgrp`

## Messaging

- SQS Queue: `sqs`
- SNS Topic: `sns`
- SNS Subscription: `snssub`

## Monitoring

- CloudWatch Log Group: `cwlog`
- CloudWatch Alarm: `cwalarm`
- CloudWatch Dashboard: `cwdash`

## Config / Secrets

- SSM Parameter: `ssmpar`
- SSM Document: `ssmdoc`
- KMS Key: `kms`
- KMS Alias: `kmsalias`
- Secrets Manager Secret: `secret`

## Random / Utility

- Random Password: `randpw`

## CDN

- CloudFront Distribution: `cfdist`
- CloudFront Cache Policy: `cfcache`
- CloudFront Function: `cffn`
- CloudFront Origin Access Control: `cfoac`
- CloudFront Response Headers Policy: `cfheaders`

## DNS

- Route 53 Hosted Zone: `r53zone`
- Route 53 Record: `r53record`
- Route 53 Health Check: `r53hc`

## ACM

- ACM Certificate: `acm`
- ACM Certificate Validation: `acmval`

## Cognito

- Cognito User Pool: `cogup`
- Cognito User Pool Client: `cogupc`
- Cognito Identity Pool: `cogip`

## SES

- SES Email Identity: `sesid`
- SES Configuration Set: `sesconfig`

## WAF

- WAF Web ACL: `wafacl`
- WAF IP Set: `wafipset`
- WAF Rule Group: `wafrg`

## CI/CD

- CodeStar Connection: `codeconn`
- CodePipeline: `pipeline`
- CodeBuild Project: `codebuild`

## Storage

- EFS File System: `efs`

## Image Builder

- Image Builder Distribution Configuration: `imgdist`
- Image Builder Component: `imgcomp`
- Image Builder Image Recipe: `imgrecipe`
- Image Builder Infrastructure Configuration: `imginfra`
- Image Builder Image: `imgbuild`

## Streaming

- Kinesis Stream: `kinesis`
- Kinesis Firehose: `firehose`

## Analytics

- Glue Catalog Database: `gluedb`
- Glue Crawler: `gluecrawler`
- Glue Job: `gluejob`
- Athena Workgroup: `athena`
- Redshift Cluster: `redshift`

## Kafka

- MSK Cluster: `msk`

## Security

- GuardDuty Detector: `guardduty`
- CloudTrail: `cloudtrail`

## Backup

- Backup Vault: `backupvault`
- Backup Plan: `backupplan`
