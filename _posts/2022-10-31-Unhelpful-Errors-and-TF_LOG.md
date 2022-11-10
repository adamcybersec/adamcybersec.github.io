---
layout: post
title: Red Herring Errors and TF LOG
tags: terraform errors troubleshooting
---

I find Terraform errors to be generally very useful. The error descriptions you get from an `init`, `validate` or `plan` are usually clear and concice, telling you exactly what is broken. I also tend to use tflint consistently to uplift the overall quality and consistency of my code (custom TFLint rules in a future post).

However, I have run into a handful of azurerm resources lately that for whatever reason, output a generic, unhelpful error description. Perhaps because the provider resource, or a feature I'm using is still too new or as a result of backend Azure API changes since the provider was last updated -  in this case, the [Windows](https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs/resources/windows_function_app) and [Linux](https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs/resources/linux_function_app) Function Apps:

```shell
2022-09-28T14:59:11.139+1000 [ERROR] vertex "module.app_service_funcapp_linux.azurerm_linux_function_app.this[\"fn01\"]" error: updating Linux Function App: (Site Name "fn01" / Resource Group "rg01"): web.AppsClient#CreateOrUpdate: Failure sending request: StatusCode=0 -- Original Error: autorest/azure: Service returned an error. Status=<nil> <nil>
```

So we're clear on exactly which instance of the resource is having issues, but `Error: autorest/azure: Service returned an error. Status=<nil> <nil>` is not very helpful.
 
I needed to dig deeper into the Terraform logs, time to jump down the rabbit hole.

Enabling Terraform Logs is easy enough using the Hashicorp documentation. If we set the `TF_LOG` environment variable to one of TRACE, DEBUG, INFO, WARN or ERROR - the verbosity of logs that output to the CLI on `stderr` changes dramatically. In reality, this was unusable for me for anything above WARN and ERROR. The volume of logging output is just silly.

> Note: I did not explore the JSON encoding of the log output with `TF_LOG=JSON` but it looks interesting enough I might write about it in the future!

Thus, we need to log out to a file that we can read later. We can do this simply by setting the `TF_LOG_PATH` environment variable to a folder path on our local system.

On my Macbook, using bash in my case:

```shell
export TF_LOG=TRACE
export TF_LOG_PATH=/tmp/log/tf-trace.log
```

Alternatively, if you use PowerShell:
```powershell
$Env:TF_LOG = "TRACE"
$Env:TF_LOG_PATH = "C:\Terraform\tf-trace.log"
```

After running `terraform apply` again, the tf-trace.log file is generated - noting that for my handful of data sources and resources, Terraform generated over 40,000 lines of log with the log level set to TRACE. Realistically, to use this file I needed to delete it or rename it after each run. We can search for the offending resource `"module.app_service_funcapp_linux.azurerm_linux_function_app.this["fn01"]"` and quickly identify some more interesting detail about our error:

```shell
{{"Code":"Conflict","Message":"Adding this VNET would exceed the App Service Plan VNET limit of 1.","Target":null,"Details":[{"Message":"Adding this VNET would exceed the App Service Plan VNET limit of 1."}},"Innererror":null}: timestamp=2022-09-28T14:59:11.110+1000
```

Great, so now we know we've got an issue with our App Service [VNET Integration](https://learn.microsoft.com/en-us/azure/app-service/overview-vnet-integration) and the error is again, abundantly clear, we've already got a VNET integrated with the App Service Plan we're trying to deploy to. So our options are either:
* Deploy this app to a different App Service Plan
* Change the VNET we're attempting to integrate with to match the existing ASP integrated VNET
* Don't use VNET Integration

In this case, my use case dictated that we spin up a new App Service Plan, integrate it with the intended VNET and then complete the deployment of my Linux Function App. 

Running `terraform apply` again, I run into the same unhelpful error - but I'm sure I fixed the VNET issue already... upon inspection of the `tf-trace.log` again, we see something different this time:

```shell
'{{"Details":[{"Message":"Subnet egress-snet in VNET vnet01 is already occupied by service /subscriptions/abcdefg-abcd-abcd-abcd-abcdef/resourceGroups/rg01/providers/Microsoft.Web/serverfarms/asp01."}]},"Innererror":null}: timestamp=2022-09-28T14:45:02.579+1000'
```

Again we see a whole lot more verbosity from the TRACE that tells us exactly what is happening. The [Subnet delegation](https://learn.microsoft.com/en-us/azure/virtual-network/subnet-delegation-overview) can only be linked to a single service. Easily fixed.


Now from a few minutes of looking at the [Hashicorp Doco](https://developer.hashicorp.com/terraform/internals/debugging) and investigating our Terraform logs we have learned three things:
1. [VNET Integration](https://learn.microsoft.com/en-us/azure/app-service/overview-vnet-integration) must use the same VNET per App Service Plan and all the child apps of that Plan :facepalm:
2. [Subnet delegation](https://learn.microsoft.com/en-us/azure/virtual-network/subnet-delegation-overview) only allows for a subnet to host a single service 
3. How to enable [Terraform Logs](https://developer.hashicorp.com/terraform/internals/debugging) using TF_LOG to get additional detail from Terraform beyond what is surfaced to the `stdout` / `stderr`

Another simple learning but a helpful one for me to not be lazy, look at the logs and find our answer quicker. TF_LOG is definitely worth turning on when you are unclear why your code is breaking or when the default cli output is being unhelpful.

Happy Clouding,

### Contact

- [LinkedIn](https://www.linkedin.com/in/adamcybersec/)<br>
- [GitHub](https://github.com/adamcybersec/)<br>
- [Email](mailto:github@adamcybersec.com)