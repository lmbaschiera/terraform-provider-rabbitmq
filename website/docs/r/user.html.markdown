---
layout: "rabbitmq"
page_title: "RabbitMQ: rabbitmq_user"
sidebar_current: "docs-rabbitmq-resource-user"
description: |-
  Creates and manages a user on a RabbitMQ server.
---

# rabbitmq\_user

The ``rabbitmq_user`` resource creates and manages a user.

~> **Note:** The `password` argument is stored in the raw state as plain-text.
Use the write-only `password_wo` argument instead to keep the password out of state
entirely. [Read more about sensitive data in state](https://developer.hashicorp.com/terraform/language/manage-sensitive-data).

## Example Usage

```hcl
resource "rabbitmq_user" "test" {
  name     = "mctest"
  password = "foobar"
  tags     = ["administrator", "management"]
}
```

### Write-only password

`password_wo` is a [write-only argument](https://developer.hashicorp.com/terraform/language/manage-sensitive-data/write-only):
Terraform passes it to the provider but never writes it to plan or state.

```hcl
resource "rabbitmq_user" "test" {
  name                = "mctest"
  password_wo         = var.rabbitmq_password
  password_wo_version = 1
  tags                = ["administrator", "management"]
}
```

To rotate the password, change `password_wo` **and** increment `password_wo_version`.

## Argument Reference

The following arguments are supported:

* `name` - (Required) The name of the user.

* `password` - (Optional) The password of the user. The value of this argument is
  plain-text and is stored in state, so make sure to secure where this is defined.
  Exactly one of `password` and `password_wo` must be set.

* `password_wo` - (Optional, Write-only) The password of the user, passed to RabbitMQ
  without ever being written to plan or state. Requires Terraform 1.11 or later, and
  must be set together with `password_wo_version`. Exactly one of `password` and
  `password_wo` must be set.

  ~> **Note:** Neither password argument may be empty. RabbitMQ does not reject a user
  that has no password: on creation it stores a random, unusable password hash, and on
  update it clears the hash entirely, wiping a previously working password. Either way
  the account can no longer authenticate, so the provider rejects an empty password
  rather than letting the apply report success.

* `password_wo_version` - (Optional) An integer that triggers a password update when
  changed. Must be set together with `password_wo`.

  ~> **Important:** Because `password_wo` is never stored in state, Terraform cannot
  detect that its value changed. Editing `password_wo` on its own **produces no plan**
  and the password is **not** rotated. You must also change `password_wo_version`.

* `tags` - (Optional) Which permission model to apply to the user. Valid
  options are: management, policymaker, monitoring, and administrator.

## Attributes Reference

No further attributes are exported.

## Import

Users can be imported using the `name`, e.g.

```
terraform import rabbitmq_user.test mctest
```

Neither `password` nor `password_wo` can be read back from RabbitMQ, so the password is
not populated by an import. When using `password_wo`, the first plan after an import
sets `password_wo_version`, which triggers an update that applies the configured password.

~> **Important:** That update overwrites whatever password the imported user already
had. When adopting an existing user, make sure the configured password is the one you
want the user to end up with.
