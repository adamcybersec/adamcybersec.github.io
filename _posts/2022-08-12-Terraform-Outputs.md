### Terraform Outputs with count and for_each
As a relative newcomer to Terraform, some rather simple things have tripped me up along my journey as I've dug around the interwebs to find the right syntax, and in some cases have stumbled across multiple that work to achieve the same effect.

Following on from my previous post about [Feature Flags]({% link _posts/2022-08-08-Terraform-Feature-Flags.md %}) I wanted to document some of the different ways to assign output values from iterated resources, generated using ```count``` or ```for_each```.

## count

The syntax options for count seem to be simply ```value = random_string.this.*.id``` or ```value = random_string.this[*].id```

**Example:**

```yaml
resource "random_string" "this" {
  count            = 1
  length           = 8
  min_special      = 8
  override_special = "$"
}

output "random_string_this_id" {
  value = random_string.this.*.id
}

# outputs:
# random_string_this_id = [
#   "$$$$$$$$",
# ]
```

or

```yaml
output "random_string_this_id" {  
  value = random_string.this[*].id
}
```


## for_each

The syntax options for for_each are a little different, where we need to extract the values using ```values(random_string.this)``` and, similar to ```count```, with either ```*``` or ```[*]```. If you prefer to stick with a 'for' syntax we can also do something that to me, feels a little more explicit: ```[ for i in random_string.this : i.id ]```

**Example:**

```yaml
variable "random_strings" {
  default = ["!", "@"]
  type    = set(string)
}

resource "random_string" "this" {
  for_each = var.random_strings

  length           = 8
  min_special      = 8
  override_special = each.value
}

output "random_string_this_id" {
  value = values(random_string.this).*.id
}

# outputs:
# random_string_this_id = [
#   "!!!!!!!!",
#   "@@@@@@@@",
# ]
```

or

```yaml
output "random_string_this_id" {
  value = values(random_string.this)[*].id
}
```

or

```yaml
output "random_string_this_id" {
  value = [ for i in random_string.this : i.id ]
}
```

Again, nothing ground breaking by any means and I'm sure there is more I could talk to but hopefully the code examples will save someone some time googling and tinkering.

Happy Clouding,

### Contact

- [LinkedIn](https://www.linkedin.com/in/adamcybersec/)<br>
- [GitHub](https://github.com/adamcybersec/)<br>
- [Email](mailto:github@adamcybersec.com)