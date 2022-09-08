---
layout: post
title: Feature Flags with Terraform
tags: terraform count for_each
---

### Fun with (Feature) Flags 
#### How to enable or disable Terraform resources with an Input Variable
We needed to be able to enable or disable individual resources within our custom Terraform modules.

For example, we created a custom module to deploy an [Azure Service Bus](https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs/resources/servicebus_namespace) that also included Service Bus [Queues](https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs/resources/servicebus_queue) and [Topics](https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs/resources/servicebus_topic).

To update or deploy Queues and Topics optionally with both existing in the same module, we needed to be able to configure our resources with [Feature Flag](https://www.atlassian.com/continuous-delivery/principles/feature-flags) functionality.


A quick Google turned up two methods, one using ```count``` and the other using ```for_each```:

#### count
If we just need to enable or disable a single instance of a resource, a simple ```bool``` does the trick:

```yaml
variable "random_string_enabled" {
  default = true
  type    = bool
}

resource "random_string" "this" {
  count            = var.random_string_enabled == true ? 1 : 0
  length           = 8
  min_special      = 8
  override_special = "$"
}
```

#### for_each
While the count solution uses a ```bool``` to achieve the desired effect well enough, there are a couple of considerations that lead me to prefer for_each:

1. We may need to pass in a ```set(string)``` with multiple values at some stage to deploy multiple instances of a resource
2. The ```count``` solution relies on the index of an unordered list so if we are ever creating multiple instances of a resource using ```count``` there is risk of [unintended results](https://www.terraform.io/language/meta-arguments/count#when-to-use-for_each-instead-of-count)
3. for_each can be used to handle a wider array of use cases including single instance and multiple instances with the same or different configurations.

```yaml
variable "random_string_enabled" {
  default = ["!", "@"]
  type    = set(string)
}

resource "random_string" "this" {
  for_each = var.random_string_enabled

  length           = 8
  min_special      = 8
  override_special = each.value
}
```

An alternative ```for_each``` syntax that is also valid for a single instance of a resource is ```for_each = var.random_string_enabled ? { enabled = true } : {}``` which is something I had not come across before.

Courtesy of [@Brendan Thompson](https://brendanthompson.com/) at [@Azenix](https://www.azenix.com.au/) I can share this explanation of how the ```{ enabled = true }``` works with ```for_each```:

*When ```var.random_string_enabled``` is true the ```for_each``` loop will use a map that contains a single key:value pair. That pair is key=enabled, value=true which produces us an instance of an object like so: ```scratch_string.this["enabled"]```.  We are able to access the ```each.*``` properties but they are essentially useless in this [single instance] scenario. When ```var.random_string_enabled``` is false what we get is an empty map which the ```for_each``` looks at and sees it has nothing to deploy.*

```yaml
variable "random_string_enabled" {
  default = true
  type    = bool
}

resource "random_string" "this" {
  for_each = var.random_string_enabled ? { enabled = true } : {}

  length           = 8
  min_special      = 8
  override_special = each.value
}
```

What if we want to main separate feature flag and argument vars ```var.random_string_enabled``` vs ```var.random_strings``` (unlike the first example) to deploy multiple instances of a resource?

Here we can pass in a ```set(string)``` for the true (enabled) expression and reference ```each.value``` as normal.

Note: ```list(string)``` doesn't work here. If you have a list input var for some reason, you'll need to use ```toset()``` to convert the incoming values:   ```for_each = var.random_string_enabled ? toset(var.random_strings) : []```

```yaml
variable "random_string_enabled" {
  default = true
  type    = bool
}

variable "random_strings" {
  default = ["!", "@"]
  type    = set(string)
}

resource "random_string" "this" {
  for_each = var.random_string_enabled ? var.random_strings : []

  length           = 8
  min_special      = 8
  override_special = each.value
}
```

While this may not be new information to many Terraform engineers out there, hopefully it will save someone a few minutes of digging.

Happy Clouding,

### Contact

- [LinkedIn](https://www.linkedin.com/in/adamcybersec/)<br>
- [GitHub](https://github.com/adamcybersec/)<br>
- [Email](mailto:github@adamcybersec.com)