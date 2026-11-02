## variables.tf

---
```hcl
variable "region" {
  type    = string
  default = "[リージョン]"
}
variable "project" {
  type    = string
  default = ”ai-content-platform"
}
variable "environment" {
  type    = string
  default = "dev"
}
variable "vpc_cidr" {
  type    = string
  default = "[IPアドレス]/16"
}
variable "enable_nat" {
  type    = bool
  default = true
}
variable "enable_aurora" {
  type    = bool
  default = false
}
variable "enable_cloudfront" {
  type    = bool
  default = false
}
variable "container_image" {
  type    = string
  default = "public.ecr.aws/docker/library/nginx:alpine"
}
variable "domain_name" {
  type        = string
  default     = ""
  description = "Optional existing Route53 zone name, e.g. example.com"
}
variable "alarm_email" {
  type    = string
  default = ""
}
variable "enable_backup" {
  type    = bool
  default = true
}
variable "enable_daily_backup" {
  type    = bool
  default = true
}
variable "enable_weekly_backup" {
  type    = bool
  default = true
}
variable "backup_retention_days" {
  type    = number
  default = 7
}
variable "backup_retention_weeks" {
  type    = number
  default = 28
}
variable "backup_retention_months" {
  type    = number
  default = 365
}
variable "enable_config" {
  type    = bool
  default = false
}
```
