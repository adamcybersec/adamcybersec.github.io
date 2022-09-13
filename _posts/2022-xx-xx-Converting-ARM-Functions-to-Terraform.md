# Blog Posts

## Converting ARM Functions to Terraform

https://www.terraform.io/language/functions/lower
https://www.terraform.io/language/functions/basename

variable "branchName" {
  type        = string
  default     = lower(basename(var.branchPath))
  # "branchName": "[toLower(last(split(parameters('branchPath'), '/')))]",
  description = "xxx"
}

https://RoddicA_P@dev.azure.com/qdigitalcode/Terraform/_git/terraform-azurerm-app-deployment