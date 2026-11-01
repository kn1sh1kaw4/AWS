## outputs.tf
```hcl
output "region" {
  value = var.region
}
output "vpc_id" {
  value = aws_vpc.main.id
}
output "alb_dns" {
  value = aws_lb.app.dns_name
}
output "web_service" {
  value = aws_ecs_service.app.name
}
output "api_service" {
  value = aws_ecs_service.api.name
}
output "cloudfront_domain" {
  value = var.enable_cloudfront ? aws_cloudfront_distribution.app[0].domain_name : null
}
output "s3_bucket" {
  value = aws_s3_bucket.content.bucket
}
output "dynamodb_table" {
  value = aws_dynamodb_table.metadata.name
}
output "ecr_repository" {
  value = aws_ecr_repository.app.repository_url
}
output "sqs_queue" {
  value = aws_sqs_queue.generation.url
}
output "step_functions" {
  value = aws_sfn_state_machine.generation.name
}
output "lambda_function" {
  value = aws_lambda_function.worker.function_name
}
output "aurora_endpoint" {
  value = var.enable_aurora ? aws_rds_cluster.aurora[0].endpoint : null
}
```
### 13. output "aurora_endpoint" {value = var.enable_aurora ? aws_rds_cluster.aurora[0].endpoint : null}
　条件分岐（三項演算子）を使用して、Auroraクラスターの接続エンドポイントをリファレンス（出力）として定義しているものです。

