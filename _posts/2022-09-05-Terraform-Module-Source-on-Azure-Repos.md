---
layout: post
title: Terraform Module Sources on Azure Repos
---

Much of the documentation and discussion around Terraform, Terraform Modules and CICD Pipelines is written, understandably, about the free and open source tools like GitHub, Gitlab, Jenkins. With my focus on the Microsoft Azure platform, occasionally I find that some rather simple tasks often blow out into a multi-hour discovery effort.

One such effort, should have been a 5 minute job, was to set our internal Terraform modules to use git as the source. In our case, git is Azure Repos on Azure DevOps.

To keep a long story short, there were a few things that needed to be done to achieve this:

### 1. Configure Azure Pipeline Permissions Project Settings
The pipelines need permission to access our Azure Repos without resorting to messy things like PATs in our pipelines and SSH Keys on our build agents.

Azure DevOps provides a better way. But first, we need our Repo permissions to be configured to allow the build agent service to authenticate to git.

We can either give permissions the `Project Collection Build Service (<ADO Organisation Name>)`, or our specific `<ProjectName> Build Service`. In this case, my project is simply named Terraform, so I have configured `Terraform Build Service (MyOrganisation)`

Go here: `https://dev.azure.com/<YOUR_ADO_ORG>/<YOUR_ADO_ORG>/_settings/repositories?_a=permissions`

Give your Build Service(s) `Contribute` permissions. You could probably get by with Read permissions, in our case we are looking to use tools like GitVersion to manage our module versioning automatically from our CICD pipelines.

![azure-devops-project-settings-repo-protection-for-yaml-pipelines]({{ site.baseurl }}/assets/img/azure-repos-permissions-for-terraform.png)

### 2. Configure Azure DevOps Project Settings
Next we need to flick a switch in the Azure DevOps Project Settings to disable the protection of Azure Repos from our Pipelines. which took my far longer to find than it should have.

Go here: `https://dev.azure.com/<YOUR_ADO_ORG>/<YOUR_ADO_ORG>/_settings/settings`

![azure-devops-project-settings-repo-protection-for-yaml-pipelines]({{ site.baseurl }}/assets/img/azure-devops-repo-pipeline-protection.png)

### 3. Persist Git Credentials in the YAML Pipeline
Here we add `persistCredentials` to our `checkout: self` task.

Potentially this step might not be essential - but for me it was when I was first getting this going. I'll set this all up from scratch again shortly and update the post if anything changes. For now, I seem to need to add the `extraheader` command to my git config:

```yaml
- checkout: self
  persistCredentials: 'true'

- script: 'git config --global http.https://<MY_ADO_ORG>@dev.azure.com/<MY_ADO_ORG>/<MY_ADO_PROJECT>.extraheader "AUTHORIZATION: bearer $(System.AccessToken)"'
```

### 4. Add the YAML Pipeline Repository Resource Definition
From the above task, note the tooltip which mentions the thing that we want most in our lives which is the 'job access token' for 'repositories that are explicitly referenced in the YAML pipeline':

![azure-devops-project-settings-tooltip]({{ site.baseurl }}/assets/img/protect-access-repos-pipelines-tooltip.png)

This is how we do that in our Azure DevOps YAML pipeline as a [Repository Resource Definition](https://docs.microsoft.com/en-us/azure/devops/pipelines/repos/multi-repo-checkout?view=azure-devops#repository-resource-definition):

```yaml
resources:
  repositories:
    - repository: MyModuleRepo
      type: git
      ref: refs/heads/main
      name: MyADOProject/terraform-azurerm-great-module
```

Note that we don't need or want to actually do a [multi-repo checkout](https://docs.microsoft.com/en-us/azure/devops/pipelines/repos/multi-repo-checkout?view=azure-devops#repository-resource-definition) which would `git clone` our module repo locally onto the agent. It is enough that we declare the Repository Resource definition to get that job access token.

### 5. Configure the Module Source in our Terraform code
Finally, the thing we actually care about.

We simply add the Azure Repos git URL to the module source:

```shell
module "great_module" {
  source = "git::https://<MY_ADO_ORG>@dev.azure.com/<MY_ADO_ORG>/<MY_ADO_PROJECT>/_git/terraform-azure-great-module"

  my_var = "some_value"
}
```

### 5. Test it in Azure Pipelines
Run your pipeline with `terraform init` and check to make sure the modules are downloaded successfully

![terraform-init-successful-module-download]({{ site.baseurl }}/assets/img/terraform-init-successful-module-download.png)


### References
It would be remiss of me not to call out the two articles that ticked most of the boxes for me here:
- [Tim Schaeps](https://www.timschaeps.be/post/dealing-with-error-tf401019-submodules-azure-pipelines/)
- [Shane Mitchell](https://www.linkedin.com/pulse/azure-devops-terrafom-introduction-shane-mitchell/)

### Epilogue
I definitely have a bad habit of skim reading documentation looking for the example code to answer my questions and maybe this is all easy to find with a little patience instead of jumping at warp speed from tab to tab but again, hopefully this post will save someone some time. If it did, shoot me a message and tell me how you spent the time you saved, hopefully with your kids, your cat, or getting in some precious, precious gaming time.

Happy Clouding,

**Contact**

- [LinkedIn](https://www.linkedin.com/in/adamcybersec/)
- [GitHub](https://github.com/adamcybersec/)
- [Email](mailto:github@adamcybersec.com)