---
layout: post
title: Enabling Azure Monitor Change Analysis with Terraform
tags: terraform azure 'azure monitor'
---

**Azure Monitor** recently introduced a brilliant new [Change Analysis](https://learn.microsoft.com/en-us/azure/azure-monitor/change/change-analysis) feature allowing us to track any changes to the App configuration, hopefully reducing the investigation time when your App experiences downtime. Microsoft highlights the following
- Azure Resource Manager resource properties
  e.g.g - Hostnames, Managed Identities
- Resource configuration changes
  - e.g. TLS settings, extension versions
- App Service / Function App in-guest changes
  - e.g. Environment variables, config files and WebJobs


### Enabling Change Analysis in the Azure Portal
Change Analysis is enabled or disabled per App or App Service Plan in the portal fairly easily, fine for a few apps, not fine for scale:

![azure-monitor-change-analysis-portal]({{ site.baseurl }}/assets/img/azure-monitor-change-analysis-portal.png)


### Enabling Change Analysis via the Azure CLI
Microsoft provides this simple set of [Azure CLI commands](https://learn.microsoft.com/en-us/azure/azure-monitor/change/change-analysis-enable#run-the-following-script) to enable Change Analysis for all your Web Apps which shows us all we need is a simple hidden tag.


### Enabling Change Analysis with Terraform
Noting that the [Web App](https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs/resources/windows_web_app) / [Function App](https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs/resources/windows_function_app) Terraform providers do not yet support enabling Change Analysis natively, we can add a very simple hidden tag, per the [Azure CLI method provided by Microsoft](https://learn.microsoft.com/en-us/azure/azure-monitor/change/change-analysis-enable#run-the-following-script), to enable Change Analysis until we are blessed by the Provider gods at which this post will be rendered obsolete.

> Note: Change Analysis currently only works for Windows based Apps and App Service Plans.

**locals.tf**
```yaml
locals {
  tags = { "hidden-related:diagnostics/changeAnalysisScanEnabled" = true }
}
```

**main.tf**
```yaml
resource "azurerm_windows_web_app" "this" {

  name                      = "app01"
  resource_group_name       = data.azurerm_resource_group.this.name
  service_plan_id           = data.azurerm_service_plan.this.id
  tags                      = merge(local.tags, var.tags)

}
```

Super simple, but a good 5 minute discovery I thought might be worth sharing.

Happy Clouding,

### Contact

- [LinkedIn](https://www.linkedin.com/in/adamcybersec/)<br>
- [GitHub](https://github.com/adamcybersec/)<br>
- [Email](mailto:github@adamcybersec.com)