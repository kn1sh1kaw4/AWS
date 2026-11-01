# Supplied architecture → Terraform mapping

| Diagram component | Terraform file/resource |
|---|---|
| Route 53 | `main.tf` / optional `aws_route53_*` can be added for custom DNS |
| CloudFront | `main.tf` `aws_cloudfront_distribution.app` |
| WAF | `main.tf` `aws_wafv2_web_acl.cf` (us-east-1 provider) |
| VPC/Subnets/IGW/NAT | `main.tf` |
| ALB | `main.tf` |
| ECS Fargate Web | `main.tf` |
| ECS Fargate API | `api.tf` |
| Lambda | `main.tf` |
| Step Functions | `main.tf` |
| SQS | `main.tf` |
| Aurora PostgreSQL | `main.tf` (optional) |
| S3 | `main.tf` |
| DynamoDB | `main.tf` |
| IAM | `main.tf` |
| CloudWatch/Logs | `main.tf` |
| SNS | `governance.tf` |
| AWS Backup | `governance.tf` |
| CloudTrail | `main.tf` |
| VPC Flow Logs | `main.tf` |
| ECR | `main.tf` |

Organizations / Control Tower are intentionally a separate account-governance project.
