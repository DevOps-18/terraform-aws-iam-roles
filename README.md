# Terraform module: AWS IAM

[![Build Status](https://travis-ci.org/cytopia/terraform-aws-iam-roles.svg?branch=master)](https://travis-ci.org/cytopia/terraform-aws-iam-roles)
[![Tag](https://img.shields.io/github/tag/cytopia/terraform-aws-iam-roles.svg)](https://github.com/cytopia/terraform-aws-iam-roles/releases)
[![Terraform](https://img.shields.io/badge/Terraform--registry-aws--iam--roles-brightgreen.svg)](https://registry.terraform.io/modules/cytopia/iam-roles/aws/)
[![License](https://img.shields.io/badge/license-MIT-blue.svg)](https://opensource.org/licenses/MIT)

This Terraform module can create an arbitrary number of IAM users, roles and policies. Roles can additionally be created with inline policies or policy ARN's attached and with trusted
entities defined as JSON or templatable json files files. Users can also additionally be created with inline policies or policy ARN's attached as well as their access key rotation can be fully managed.


## Deprecation Warning

This repository is deprecated as all IAM features have been integrated into: [github.com/cytopia/terraform-aws-iam](https://github.com/cytopia/terraform-aws-iam)


## Important note

When creating an IAM user with an `Inactive` access key it is initially created with access key set to `Active`. You will have to run it a second time in order to deactivate the access key.
This is either an issue with the terraform resource `aws_iam_access_key` or with the AWS api itself.


## Usage

### Assumeable roles

```hcl
module "iam_roles" {
  source = "github.com/cytopia/terraform-aws-iam-roles?ref=v2.0.0"

  # List of policies to create
  policies = [
    {
      name = "ro-billing"
      path = "/assume/human/"
      desc = "Provides read-only access to billing"
      file = "policies/ro-billing.json"
      vars = {}
    },
  ]

  # List of users to manage
  users = [
    {
      name                 = "admin"
      path                 = null
      access_keys          = []
      permissions_boundary = null
      policies             = []
      inline_policies      = []
      policy_arns = [
        "arn:aws:iam::aws:policy/AdministratorAccess",
      ]
    },
    {
      name        = "developer"
      path        = null
      access_keys = [
        {
          name    = "key-1"
          pgp_key = ""
          status  = "Active"
        }
      ]
      permissions_boundary = "arn:aws:iam::aws:policy/PowerUserAccess"
      policies    = [
        "rds-authenticate",
      ]
      inline_policies = []
      policy_arns     = []
    },
  ]

  # List of roles to manage
  roles = [
    {
      name                 = "ROLE-ADMIN"
      path                 = ""
      desc                 = ""
      trust_policy_file    = "trust-policies/admin.json"
      permissions_boundary = null
      policies             = []
      inline_policies      = []
      policy_arns = [
        "arn:aws:iam::aws:policy/AdministratorAccess",
      ]
    },
    {
      name                 = "ROLE-DEV"
      path                 = ""
      desc                 = ""
      trust_policy_file    = "trust-policies/dev.json"
      permissions_boundary = "arn:aws:iam::aws:policy/PowerUserAccess"
      policies = [
        "ro-billing",
      ]
      inline_policies = []
      policy_arns = [
        "arn:aws:iam::aws:policy/PowerUserAccess",
      ]
    },
  ]

}
```

**`trust-policies/admin.json`**

Defines the permissions (Authorization)
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "sts:AssumeRole",
      "Principal": {
        "AWS": [
          "arn:aws:iam::1234567:role/federation/LOGIN-ADMIN"
        ]
      },
      "Condition": {}
    }
  ]
}
```
**`trust-policies/dev.json`**

Defines the permissions (Authorization)
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "sts:AssumeRole",
      "Principal": {
        "AWS": [
          "arn:aws:iam::1234567:role/federation/LOGIN-DEV",
          "arn:aws:iam::1234567:role/federation/LOGIN-ADMIN"
        ]
      },
      "Condition": {}
    }
  ]
}
```


**`policies/ro-billing.json`**
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "BillingReadOnly",
      "Effect": "Allow",
      "Action": [
        "account:ListRegions",
        "aws-portal:View*",
        "awsbillingconsole:View*",
        "budgets:View*",
        "ce:Get*",
        "cur:Describe*",
        "pricing:Describe*",
        "pricing:Get*"
      ],
      "Resource": "*"
    }
  ]
}
```


<!-- BEGINNING OF PRE-COMMIT-TERRAFORM DOCS HOOK -->
## Requirements

| Name | Version |
|------|---------|
| terraform | >= 0.12.6 |

## Providers

| Name | Version |
|------|---------|
| aws | n/a |

## Inputs

| Name | Description | Type | Default | Required |
|------|-------------|------|---------|:--------:|
| roles | A list of dictionaries defining all roles. | <pre>list(object({<br>    name                 = string       # Name of the role<br>    path                 = string       # Defaults to 'var.role_path' variable is set to null<br>    desc                 = string       # Defaults to 'var.role_desc' variable is set to null<br>    trust_policy_file    = string       # Path to file of trust/assume policy<br>    permissions_boundary = string       # ARN to a policy used as permissions boundary (or null/empty)<br>    policies             = list(string) # List of names of policies (must be defined in var.policies)<br>    inline_policies      = list(object({<br>      name = string      # Name of the inline policy<br>      file = string      # Path to json or json.tmpl file of policy<br>      vars = map(string) # Policy template variables {key = val, ...}<br>    }))<br>    policy_arns = list(string) # List of existing policy ARN's<br>  }))</pre> | n/a | yes |
| users | A list of dictionaries defining all users. | <pre>list(object({<br>    name = string # Name of the user<br>    path = string # Defaults to 'var.user_path' variable is set to null<br>    access_keys = list(object({<br>      name    = string # IaC identifier for first or second IAM access key (not used on AWS)<br>      pgp_key = string # Leave empty for non or provide a b64-enc pubkey or keybase username<br>      status  = string # 'Active' or 'Inactive'<br>    }))<br>    permissions_boundary = string # ARN to a policy used as permissions boundary (or null/empty)<br>    policies = list(string)       # List of names of policies (must be defined in var.policies)<br>    inline_policies = list(object({<br>      name = string      # Name of the inline policy<br>      file = string      # Path to json or json.tmpl file of policy<br>      vars = map(string) # Policy template variables {key = val, ...}<br>    }))<br>    policy_arns = list(string) # List of existing policy ARN's<br>  }))</pre> | n/a | yes |
| policies | A list of dictionaries defining all policies. | <pre>list(object({<br>    name = string      # Name of the policy<br>    path = string      # Defaults to 'var.policy_path' variable is set to null<br>    desc = string      # Defaults to 'var.policy_desc' variable is set to null<br>    file = string      # Path to json or json.tmpl file of policy<br>    vars = map(string) # Policy template variables {key: val, ...}<br>  }))</pre> | `[]` | no |
| policy\_desc | The default description of the policy. | `string` | `"Managed by Terraform"` | no |
| policy\_path | The default path under which to create the policy if not specified in the policies list. You can use a single path, or nest multiple paths as if they were a folder structure. For example, you could use the nested path /division\_abc/subdivision\_xyz/product\_1234/engineering/ to match your company's organizational structure. | `string` | `"/"` | no |
| role\_desc | The description of the role. | `string` | `"Managed by Terraform"` | no |
| role\_force\_detach\_policies | Specifies to force detaching any policies the role has before destroying it. | `bool` | `true` | no |
| role\_max\_session\_duration | The maximum session duration (in seconds) that you want to set for the specified role. This setting can have a value from 1 hour to 12 hours specified in seconds. | `string` | `"3600"` | no |
| role\_path | The path under which to create the role. You can use a single path, or nest multiple paths as if they were a folder structure. For example, you could use the nested path /division\_abc/subdivision\_xyz/product\_1234/engineering/ to match your company's organizational structure. | `string` | `"/"` | no |
| tags | Key-value mapping of tags for the IAM role or user. | `map(any)` | `{}` | no |
| user\_path | The path under which to create the user. You can use a single path, or nest multiple paths as if they were a folder structure. For example, you could use the nested path /division\_abc/subdivision\_xyz/product\_1234/engineering/ to match your company's organizational structure. | `string` | `"/"` | no |

## Outputs

| Name | Description |
|------|-------------|
| debug\_local\_policies | The transformed policy map |
| debug\_local\_role\_inline\_policies | The transformed role inline policy map |
| debug\_local\_role\_policies | The transformed role policy map |
| debug\_local\_role\_policy\_arns | The transformed role policy arns map |
| debug\_local\_user\_access\_keys | The transformed user access key map |
| debug\_local\_user\_inline\_policies | The transformed user inline policy map |
| debug\_local\_user\_policies | The transformed user policy map |
| debug\_local\_user\_policy\_arns | The transformed user policy arns map |
| debug\_var\_policies | The transformed policy map |
| debug\_var\_roles | The defined roles list |
| debug\_var\_users | The defined users list |
| policies | Created customer managed IAM policies |
| role\_inline\_policy\_attachments | Attached role inline IAM policies |
| role\_policy\_arn\_attachments | Attached role IAM policy arns |
| role\_policy\_attachments | Attached role customer managed IAM policies |
| roles | Created IAM roles |
| user\_inline\_policy\_attachments | Attached user inline IAM policies |
| user\_policy\_arn\_attachments | Attached user IAM policy arns |
| user\_policy\_attachments | Attached user customer managed IAM policies |
| users | Created IAM users |

<!-- END OF PRE-COMMIT-TERRAFORM DOCS HOOK -->

## Authors

Module managed by [cytopia](https://github.com/cytopia).


## License

[MIT License](LICENSE)

Copyright (c) 2018 [cytopia](https://github.com/cytopia)
