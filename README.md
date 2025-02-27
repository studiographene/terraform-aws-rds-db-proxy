# Purpose

Module Courtesy: **cloudposse**

Terraform module to provision an Amazon [RDS Proxy](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/rds-proxy.html) for MySQL or Postgres.

---

It's 100% Open Source and licensed under the [APACHE2](LICENSE).

## Security & Compliance [<img src="https://cloudposse.com/wp-content/uploads/2020/11/bridgecrew.svg" width="250" align="right" />](https://bridgecrew.io/)

Security scanning is graciously provided by Bridgecrew. Bridgecrew is the leading fully hosted, cloud-native solution providing continuous Terraform security and compliance.

| Benchmark                                                                                                                                                                                                                                                     | Description                                                      |
| ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------- |
| [![Infrastructure Security](https://www.bridgecrew.cloud/badges/github/cloudposse/terraform-aws-rds-db-proxy/general)](https://www.bridgecrew.cloud/link/badge?vcs=github&fullRepo=cloudposse%2Fterraform-aws-rds-db-proxy&benchmark=INFRASTRUCTURE+SECURITY) | Infrastructure Security Compliance                               |
| [![CIS KUBERNETES](https://www.bridgecrew.cloud/badges/github/cloudposse/terraform-aws-rds-db-proxy/cis_kubernetes)](https://www.bridgecrew.cloud/link/badge?vcs=github&fullRepo=cloudposse%2Fterraform-aws-rds-db-proxy&benchmark=CIS+KUBERNETES+V1.5)       | Center for Internet Security, KUBERNETES Compliance              |
| [![CIS AWS](https://www.bridgecrew.cloud/badges/github/cloudposse/terraform-aws-rds-db-proxy/cis_aws)](https://www.bridgecrew.cloud/link/badge?vcs=github&fullRepo=cloudposse%2Fterraform-aws-rds-db-proxy&benchmark=CIS+AWS+V1.2)                            | Center for Internet Security, AWS Compliance                     |
| [![CIS AZURE](https://www.bridgecrew.cloud/badges/github/cloudposse/terraform-aws-rds-db-proxy/cis_azure)](https://www.bridgecrew.cloud/link/badge?vcs=github&fullRepo=cloudposse%2Fterraform-aws-rds-db-proxy&benchmark=CIS+AZURE+V1.1)                      | Center for Internet Security, AZURE Compliance                   |
| [![PCI-DSS](https://www.bridgecrew.cloud/badges/github/cloudposse/terraform-aws-rds-db-proxy/pci)](https://www.bridgecrew.cloud/link/badge?vcs=github&fullRepo=cloudposse%2Fterraform-aws-rds-db-proxy&benchmark=PCI-DSS+V3.2)                                | Payment Card Industry Data Security Standards Compliance         |
| [![NIST-800-53](https://www.bridgecrew.cloud/badges/github/cloudposse/terraform-aws-rds-db-proxy/nist)](https://www.bridgecrew.cloud/link/badge?vcs=github&fullRepo=cloudposse%2Fterraform-aws-rds-db-proxy&benchmark=NIST-800-53)                            | National Institute of Standards and Technology Compliance        |
| [![ISO27001](https://www.bridgecrew.cloud/badges/github/cloudposse/terraform-aws-rds-db-proxy/iso)](https://www.bridgecrew.cloud/link/badge?vcs=github&fullRepo=cloudposse%2Fterraform-aws-rds-db-proxy&benchmark=ISO27001)                                   | Information Security Management System, ISO/IEC 27001 Compliance |
| [![SOC2](https://www.bridgecrew.cloud/badges/github/cloudposse/terraform-aws-rds-db-proxy/soc2)](https://www.bridgecrew.cloud/link/badge?vcs=github&fullRepo=cloudposse%2Fterraform-aws-rds-db-proxy&benchmark=SOC2)                                          | Service Organization Control 2 Compliance                        |
| [![CIS GCP](https://www.bridgecrew.cloud/badges/github/cloudposse/terraform-aws-rds-db-proxy/cis_gcp)](https://www.bridgecrew.cloud/link/badge?vcs=github&fullRepo=cloudposse%2Fterraform-aws-rds-db-proxy&benchmark=CIS+GCP+V1.1)                            | Center for Internet Security, GCP Compliance                     |
| [![HIPAA](https://www.bridgecrew.cloud/badges/github/cloudposse/terraform-aws-rds-db-proxy/hipaa)](https://www.bridgecrew.cloud/link/badge?vcs=github&fullRepo=cloudposse%2Fterraform-aws-rds-db-proxy&benchmark=HIPAA)                                       | Health Insurance Portability and Accountability Compliance       |

## Usage

**IMPORTANT:** We do not pin modules to versions in our examples because of the
difficulty of keeping the versions in the documentation in sync with the latest released versions.
We highly recommend that in your code you pin the version to the exact version you are
using so that your infrastructure remains stable, and update versions in a
systematic way so that they do not catch you by surprise.

Also, because of a bug in the Terraform registry ([hashicorp/terraform#21417](https://github.com/hashicorp/terraform/issues/21417)),
the registry shows many of our inputs as required when in fact they are optional.
The table below correctly indicates which inputs are required.

For a complete example, see [examples/complete](examples/complete).

For automated tests of the complete example using [bats](https://github.com/bats-core/bats-core) and [Terratest](https://github.com/gruntwork-io/terratest)
(which tests and deploys the example on AWS), see [test](test).

```hcl
module "network" {
  source  = "app.terraform.io/studiographene/network/aws"
  # version = "x.x.x"

  context = module.this.context

  cidr_block = "172.16.0.0/16"
  nat_gateway_enabled  = false

}

resource "random_password" "admin_password" {
  count  = var.database_password == "" || var.database_password == null ? 1 : 0
  length           = 33
  special          = false
  override_special = "!#$%^&*()<>-_"
}

locals {
  database_password = var.database_password != "" && var.database_password != null ? var.database_password : join("", random_password.admin_password.*.result)

  username_password = {
    username = var.database_user
    password = local.database_password
  }

  auth = [
    {
      auth_scheme = "SECRETS"
      description = "Access the database instance using username and password from AWS Secrets Manager"
      iam_auth    = "DISABLED"
      secret_arn  = aws_secretsmanager_secret.rds_username_and_password.arn
    }
  ]
}

module "rds_instance" {
  source  = "app.terraform.io/studiographene/rds/aws"
  # version = "x.x.x"

  database_name       = var.database_name
  database_user       = var.database_user
  database_password   = local.database_password
  database_port       = var.database_port
  multi_az            = var.multi_az
  storage_type        = var.storage_type
  allocated_storage   = var.allocated_storage
  storage_encrypted   = var.storage_encrypted
  engine              = var.engine
  engine_version      = var.engine_version
  instance_class      = var.instance_class
  db_parameter_group  = var.db_parameter_group
  publicly_accessible = var.publicly_accessible
  vpc_id              = module.network.vpc_id
  subnet_ids          = module.subnets.private_subnet_ids
  security_group_ids  = ["sg-1234545abc"]
  apply_immediately   = var.apply_immediately

  context = module.this.context
}

resource "aws_secretsmanager_secret" "rds_username_and_password" {
  name                    = module.this.id
  description             = "RDS username and password"
  recovery_window_in_days = 0
  tags                    = module.this.tags
}

resource "aws_secretsmanager_secret_version" "rds_username_and_password" {
  secret_id     = aws_secretsmanager_secret.rds_username_and_password.id
  secret_string = jsonencode(local.username_password)
}

module "rds_proxy" {
  source  = "app.terraform.io/studiographene/rds-db-proxy/aws"
  # version = "x.x.x"
  db_instance_identifier = module.rds_instance.instance_id
  auth                   = local.auth
  vpc_security_group_ids = [module.network.vpc_default_security_group_id]
  vpc_subnet_ids         = module.subnets.public_subnet_ids

  debug_logging                = var.debug_logging
  engine_family                = var.engine_family
  idle_client_timeout          = var.idle_client_timeout
  require_tls                  = var.require_tls
  connection_borrow_timeout    = var.connection_borrow_timeout
  init_query                   = var.init_query
  max_connections_percent      = var.max_connections_percent
  max_idle_connections_percent = var.max_idle_connections_percent
  session_pinning_filters      = var.session_pinning_filters
  existing_iam_role_arn        = var.existing_iam_role_arn

  context = module.this.context
}
```

## Examples

Review the [complete example](examples/complete) to see how to use this module.

<!-- markdownlint-disable -->

## Makefile Targets

```text
Available targets:

  help                                Help screen
  help/all                            Display help for all targets
  help/short                          This help short screen
  lint                                Lint terraform code

```

<!-- markdownlint-restore -->
<!-- markdownlint-disable -->

## Requirements

| Name                                                                     | Version   |
| ------------------------------------------------------------------------ | --------- |
| <a name="requirement_terraform"></a> [terraform](#requirement_terraform) | >= 0.13.0 |
| <a name="requirement_aws"></a> [aws](#requirement_aws)                   | >= 3.1.15 |

## Providers

| Name                                             | Version   |
| ------------------------------------------------ | --------- |
| <a name="provider_aws"></a> [aws](#provider_aws) | >= 3.1.15 |

## Modules

| Name                                                              | Source                                        | Version |
| ----------------------------------------------------------------- | --------------------------------------------- | ------- |
| <a name="module_role_label"></a> [role_label](#module_role_label) | app.terraform.io/studiographene/sg-label/null | 1.0.4   |
| <a name="module_this"></a> [this](#module_this)                   | app.terraform.io/studiographene/sg-label/null | 1.0.4   |

## Resources

| Name                                                                                                                                                | Type        |
| --------------------------------------------------------------------------------------------------------------------------------------------------- | ----------- |
| [aws_db_proxy.this](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/db_proxy)                                           | resource    |
| [aws_db_proxy_default_target_group.this](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/db_proxy_default_target_group) | resource    |
| [aws_db_proxy_target.this](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/db_proxy_target)                             | resource    |
| [aws_iam_policy.this](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/iam_policy)                                       | resource    |
| [aws_iam_role.this](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/iam_role)                                           | resource    |
| [aws_iam_role_policy_attachment.this](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/iam_role_policy_attachment)       | resource    |
| [aws_iam_policy_document.assume_role](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/data-sources/iam_policy_document)           | data source |
| [aws_iam_policy_document.this](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/data-sources/iam_policy_document)                  | data source |
| [aws_kms_key.this](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/data-sources/kms_key)                                          | data source |
| [aws_region.this](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/data-sources/region)                                            | data source |

## Inputs

| Name                                                                                                                  | Description                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          | Type                                                                                                                             | Default                                                                                                                                                                                                                                                                                                                                                                                                                                                           | Required |
| --------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | :------: |
| <a name="input_additional_tag_map"></a> [additional_tag_map](#input_additional_tag_map)                               | Additional key-value pairs to add to each map in `tags_as_list_of_maps`. Not added to `tags` or `id`.<br>This is for some rare cases where resources want additional configuration of tags<br>and therefore take a list of maps with tag key, value, and additional configuration.                                                                                                                                                                                                                                                                                                                                                                                   | `map(string)`                                                                                                                    | `{}`                                                                                                                                                                                                                                                                                                                                                                                                                                                              |    no    |
| <a name="input_attributes"></a> [attributes](#input_attributes)                                                       | ID element. Additional attributes (e.g. `workers` or `cluster`) to add to `id`,<br>in the order they appear in the list. New attributes are appended to the<br>end of the list. The elements of the list are joined by the `delimiter`<br>and treated as a single ID element.                                                                                                                                                                                                                                                                                                                                                                                        | `list(string)`                                                                                                                   | `[]`                                                                                                                                                                                                                                                                                                                                                                                                                                                              |    no    |
| <a name="input_auth"></a> [auth](#input_auth)                                                                         | Configuration blocks with authorization mechanisms to connect to the associated database instances or clusters                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       | <pre>list(object({<br> auth_scheme = string<br> description = string<br> iam_auth = string<br> secret_arn = string<br> }))</pre> | n/a                                                                                                                                                                                                                                                                                                                                                                                                                                                               |   yes    |
| <a name="input_connection_borrow_timeout"></a> [connection_borrow_timeout](#input_connection_borrow_timeout)          | The number of seconds for a proxy to wait for a connection to become available in the connection pool. Only applies when the proxy has opened its maximum number of connections and all connections are busy with client sessions                                                                                                                                                                                                                                                                                                                                                                                                                                    | `number`                                                                                                                         | `120`                                                                                                                                                                                                                                                                                                                                                                                                                                                             |    no    |
| <a name="input_context"></a> [context](#input_context)                                                                | Single object for setting entire context at once.<br>See description of individual variables for details.<br>Leave string and numeric variables as `null` to use default value.<br>Individual variable settings (non-null) override settings in context object,<br>except for attributes, tags, and additional_tag_map, which are merged.                                                                                                                                                                                                                                                                                                                            | `any`                                                                                                                            | <pre>{<br> "additional_tag_map": {},<br> "attributes": [],<br> "delimiter": null,<br> "descriptor_formats": {},<br> "enabled": true,<br> "environment": null,<br> "id_length_limit": null,<br> "label_key_case": null,<br> "label_order": [],<br> "label_value_case": null,<br> "labels_as_tags": [<br> "unset"<br> ],<br> "name": null,<br> "namespace": null,<br> "regex_replace_chars": null,<br> "stage": null,<br> "tags": {},<br> "tenant": null<br>}</pre> |    no    |
| <a name="input_db_cluster_identifier"></a> [db_cluster_identifier](#input_db_cluster_identifier)                      | DB cluster identifier. Either `db_instance_identifier` or `db_cluster_identifier` should be specified and both should not be specified together                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      | `string`                                                                                                                         | `null`                                                                                                                                                                                                                                                                                                                                                                                                                                                            |    no    |
| <a name="input_db_instance_identifier"></a> [db_instance_identifier](#input_db_instance_identifier)                   | DB instance identifier. Either `db_instance_identifier` or `db_cluster_identifier` should be specified and both should not be specified together                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     | `string`                                                                                                                         | `null`                                                                                                                                                                                                                                                                                                                                                                                                                                                            |    no    |
| <a name="input_debug_logging"></a> [debug_logging](#input_debug_logging)                                              | Whether the proxy includes detailed information about SQL statements in its logs                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     | `bool`                                                                                                                           | `false`                                                                                                                                                                                                                                                                                                                                                                                                                                                           |    no    |
| <a name="input_delimiter"></a> [delimiter](#input_delimiter)                                                          | Delimiter to be used between ID elements.<br>Defaults to `-` (hyphen). Set to `""` to use no delimiter at all.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       | `string`                                                                                                                         | `null`                                                                                                                                                                                                                                                                                                                                                                                                                                                            |    no    |
| <a name="input_descriptor_formats"></a> [descriptor_formats](#input_descriptor_formats)                               | Describe additional descriptors to be output in the `descriptors` output map.<br>Map of maps. Keys are names of descriptors. Values are maps of the form<br>`{<br>   format = string<br>   labels = list(string)<br>}`<br>(Type is `any` so the map values can later be enhanced to provide additional options.)<br>`format` is a Terraform format string to be passed to the `format()` function.<br>`labels` is a list of labels, in order, to pass to `format()` function.<br>Label values will be normalized before being passed to `format()` so they will be<br>identical to how they appear in `id`.<br>Default is `{}` (`descriptors` output will be empty). | `any`                                                                                                                            | `{}`                                                                                                                                                                                                                                                                                                                                                                                                                                                              |    no    |
| <a name="input_enabled"></a> [enabled](#input_enabled)                                                                | Set to false to prevent the module from creating any resources                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       | `bool`                                                                                                                           | `null`                                                                                                                                                                                                                                                                                                                                                                                                                                                            |    no    |
| <a name="input_engine_family"></a> [engine_family](#input_engine_family)                                              | The kinds of databases that the proxy can connect to. This value determines which database network protocol the proxy recognizes when it interprets network traffic to and from the database. The engine family applies to MySQL and PostgreSQL for both RDS and Aurora. Valid values are MYSQL and POSTGRESQL                                                                                                                                                                                                                                                                                                                                                       | `string`                                                                                                                         | `"MYSQL"`                                                                                                                                                                                                                                                                                                                                                                                                                                                         |    no    |
| <a name="input_environment"></a> [environment](#input_environment)                                                    | ID element. Usually used for region e.g. 'uw2', 'us-west-2', OR role 'prod', 'staging', 'dev', 'UAT'                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 | `string`                                                                                                                         | `null`                                                                                                                                                                                                                                                                                                                                                                                                                                                            |    no    |
| <a name="input_existing_iam_role_arn"></a> [existing_iam_role_arn](#input_existing_iam_role_arn)                      | The ARN of an existing IAM role that the proxy can use to access secrets in AWS Secrets Manager. If not provided, the module will create a role to access secrets in Secrets Manager                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 | `string`                                                                                                                         | `null`                                                                                                                                                                                                                                                                                                                                                                                                                                                            |    no    |
| <a name="input_iam_role_attributes"></a> [iam_role_attributes](#input_iam_role_attributes)                            | Additional attributes to add to the ID of the IAM role that the proxy uses to access secrets in AWS Secrets Manager                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  | `list(string)`                                                                                                                   | `null`                                                                                                                                                                                                                                                                                                                                                                                                                                                            |    no    |
| <a name="input_id_length_limit"></a> [id_length_limit](#input_id_length_limit)                                        | Limit `id` to this many characters (minimum 6).<br>Set to `0` for unlimited length.<br>Set to `null` for keep the existing setting, which defaults to `0`.<br>Does not affect `id_full`.                                                                                                                                                                                                                                                                                                                                                                                                                                                                             | `number`                                                                                                                         | `null`                                                                                                                                                                                                                                                                                                                                                                                                                                                            |    no    |
| <a name="input_idle_client_timeout"></a> [idle_client_timeout](#input_idle_client_timeout)                            | The number of seconds that a connection to the proxy can be inactive before the proxy disconnects it                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 | `number`                                                                                                                         | `1800`                                                                                                                                                                                                                                                                                                                                                                                                                                                            |    no    |
| <a name="input_init_query"></a> [init_query](#input_init_query)                                                       | One or more SQL statements for the proxy to run when opening each new database connection                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            | `string`                                                                                                                         | `null`                                                                                                                                                                                                                                                                                                                                                                                                                                                            |    no    |
| <a name="input_kms_key_id"></a> [kms_key_id](#input_kms_key_id)                                                       | The ARN or Id of the AWS KMS customer master key (CMK) to encrypt the secret values in the versions stored in secrets. If you don't specify this value, then Secrets Manager defaults to using the AWS account's default CMK (the one named `aws/secretsmanager`)                                                                                                                                                                                                                                                                                                                                                                                                    | `string`                                                                                                                         | `null`                                                                                                                                                                                                                                                                                                                                                                                                                                                            |    no    |
| <a name="input_label_key_case"></a> [label_key_case](#input_label_key_case)                                           | Controls the letter case of the `tags` keys (label names) for tags generated by this module.<br>Does not affect keys of tags passed in via the `tags` input.<br>Possible values: `lower`, `title`, `upper`.<br>Default value: `title`.                                                                                                                                                                                                                                                                                                                                                                                                                               | `string`                                                                                                                         | `null`                                                                                                                                                                                                                                                                                                                                                                                                                                                            |    no    |
| <a name="input_label_order"></a> [label_order](#input_label_order)                                                    | The order in which the labels (ID elements) appear in the `id`.<br>Defaults to ["namespace", "environment", "stage", "name", "attributes"].<br>You can omit any of the 6 labels ("tenant" is the 6th), but at least one must be present.                                                                                                                                                                                                                                                                                                                                                                                                                             | `list(string)`                                                                                                                   | `null`                                                                                                                                                                                                                                                                                                                                                                                                                                                            |    no    |
| <a name="input_label_value_case"></a> [label_value_case](#input_label_value_case)                                     | Controls the letter case of ID elements (labels) as included in `id`,<br>set as tag values, and output by this module individually.<br>Does not affect values of tags passed in via the `tags` input.<br>Possible values: `lower`, `title`, `upper` and `none` (no transformation).<br>Set this to `title` and set `delimiter` to `""` to yield Pascal Case IDs.<br>Default value: `lower`.                                                                                                                                                                                                                                                                          | `string`                                                                                                                         | `null`                                                                                                                                                                                                                                                                                                                                                                                                                                                            |    no    |
| <a name="input_labels_as_tags"></a> [labels_as_tags](#input_labels_as_tags)                                           | Set of labels (ID elements) to include as tags in the `tags` output.<br>Default is to include all labels.<br>Tags with empty values will not be included in the `tags` output.<br>Set to `[]` to suppress all generated tags.<br>**Notes:**<br> The value of the `name` tag, if included, will be the `id`, not the `name`.<br> Unlike other `null-label` inputs, the initial setting of `labels_as_tags` cannot be<br> changed in later chained modules. Attempts to change it will be silently ignored.                                                                                                                                                            | `set(string)`                                                                                                                    | <pre>[<br> "default"<br>]</pre>                                                                                                                                                                                                                                                                                                                                                                                                                                   |    no    |
| <a name="input_max_connections_percent"></a> [max_connections_percent](#input_max_connections_percent)                | The maximum size of the connection pool for each target in a target group                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            | `number`                                                                                                                         | `100`                                                                                                                                                                                                                                                                                                                                                                                                                                                             |    no    |
| <a name="input_max_idle_connections_percent"></a> [max_idle_connections_percent](#input_max_idle_connections_percent) | Controls how actively the proxy closes idle database connections in the connection pool. A high value enables the proxy to leave a high percentage of idle connections open. A low value causes the proxy to close idle client connections and return the underlying database connections to the connection pool                                                                                                                                                                                                                                                                                                                                                     | `number`                                                                                                                         | `50`                                                                                                                                                                                                                                                                                                                                                                                                                                                              |    no    |
| <a name="input_name"></a> [name](#input_name)                                                                         | ID element. Usually the component or solution name, e.g. 'app' or 'jenkins'.<br>This is the only ID element not also included as a `tag`.<br>The "name" tag is set to the full `id` string. There is no tag with the value of the `name` input.                                                                                                                                                                                                                                                                                                                                                                                                                      | `string`                                                                                                                         | `null`                                                                                                                                                                                                                                                                                                                                                                                                                                                            |    no    |
| <a name="input_namespace"></a> [namespace](#input_namespace)                                                          | ID element. Usually an abbreviation of your organization name, e.g. 'eg' or 'cp', to help ensure generated IDs are globally unique                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   | `string`                                                                                                                         | `null`                                                                                                                                                                                                                                                                                                                                                                                                                                                            |    no    |
| <a name="input_proxy_create_timeout"></a> [proxy_create_timeout](#input_proxy_create_timeout)                         | Proxy creation timeout                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               | `string`                                                                                                                         | `"30m"`                                                                                                                                                                                                                                                                                                                                                                                                                                                           |    no    |
| <a name="input_proxy_delete_timeout"></a> [proxy_delete_timeout](#input_proxy_delete_timeout)                         | Proxy delete timeout                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 | `string`                                                                                                                         | `"60m"`                                                                                                                                                                                                                                                                                                                                                                                                                                                           |    no    |
| <a name="input_proxy_update_timeout"></a> [proxy_update_timeout](#input_proxy_update_timeout)                         | Proxy update timeout                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 | `string`                                                                                                                         | `"30m"`                                                                                                                                                                                                                                                                                                                                                                                                                                                           |    no    |
| <a name="input_regex_replace_chars"></a> [regex_replace_chars](#input_regex_replace_chars)                            | Terraform regular expression (regex) string.<br>Characters matching the regex will be removed from the ID elements.<br>If not set, `"/[^a-zA-Z0-9-]/"` is used to remove all characters other than hyphens, letters and digits.                                                                                                                                                                                                                                                                                                                                                                                                                                      | `string`                                                                                                                         | `null`                                                                                                                                                                                                                                                                                                                                                                                                                                                            |    no    |
| <a name="input_require_tls"></a> [require_tls](#input_require_tls)                                                    | A Boolean parameter that specifies whether Transport Layer Security (TLS) encryption is required for connections to the proxy. By enabling this setting, you can enforce encrypted TLS connections to the proxy                                                                                                                                                                                                                                                                                                                                                                                                                                                      | `bool`                                                                                                                           | `false`                                                                                                                                                                                                                                                                                                                                                                                                                                                           |    no    |
| <a name="input_session_pinning_filters"></a> [session_pinning_filters](#input_session_pinning_filters)                | Each item in the list represents a class of SQL operations that normally cause all later statements in a session using a proxy to be pinned to the same underlying database connection                                                                                                                                                                                                                                                                                                                                                                                                                                                                               | `list(string)`                                                                                                                   | `null`                                                                                                                                                                                                                                                                                                                                                                                                                                                            |    no    |
| <a name="input_stage"></a> [stage](#input_stage)                                                                      | ID element. Usually used to indicate role, e.g. 'prod', 'staging', 'source', 'build', 'test', 'deploy', 'release'                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    | `string`                                                                                                                         | `null`                                                                                                                                                                                                                                                                                                                                                                                                                                                            |    no    |
| <a name="input_tags"></a> [tags](#input_tags)                                                                         | Additional tags (e.g. `{'BusinessUnit': 'XYZ'}`).<br>Neither the tag keys nor the tag values will be modified by this module.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        | `map(string)`                                                                                                                    | `{}`                                                                                                                                                                                                                                                                                                                                                                                                                                                              |    no    |
| <a name="input_tenant"></a> [tenant](#input_tenant)                                                                   | ID element \_(Rarely used, not included by default)\_. A customer identifier, indicating who this instance of a resource is for                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      | `string`                                                                                                                         | `null`                                                                                                                                                                                                                                                                                                                                                                                                                                                            |    no    |
| <a name="input_vpc_security_group_ids"></a> [vpc_security_group_ids](#input_vpc_security_group_ids)                   | One or more VPC security group IDs to associate with the proxy                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       | `set(string)`                                                                                                                    | n/a                                                                                                                                                                                                                                                                                                                                                                                                                                                               |   yes    |
| <a name="input_vpc_subnet_ids"></a> [vpc_subnet_ids](#input_vpc_subnet_ids)                                           | One or more VPC subnet IDs to associate with the proxy                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               | `set(string)`                                                                                                                    | n/a                                                                                                                                                                                                                                                                                                                                                                                                                                                               |   yes    |

## Outputs

| Name                                                                                                                             | Description                                                                                                                                                              |
| -------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| <a name="output_proxy_arn"></a> [proxy_arn](#output_proxy_arn)                                                                   | Proxy ARN                                                                                                                                                                |
| <a name="output_proxy_default_target_group_arn"></a> [proxy_default_target_group_arn](#output_proxy_default_target_group_arn)    | The Amazon Resource Name (ARN) representing the default target group                                                                                                     |
| <a name="output_proxy_default_target_group_name"></a> [proxy_default_target_group_name](#output_proxy_default_target_group_name) | The name of the default target group                                                                                                                                     |
| <a name="output_proxy_endpoint"></a> [proxy_endpoint](#output_proxy_endpoint)                                                    | Proxy endpoint                                                                                                                                                           |
| <a name="output_proxy_iam_role_arn"></a> [proxy_iam_role_arn](#output_proxy_iam_role_arn)                                        | The ARN of the IAM role that the proxy uses to access secrets in AWS Secrets Manager                                                                                     |
| <a name="output_proxy_id"></a> [proxy_id](#output_proxy_id)                                                                      | Proxy ID                                                                                                                                                                 |
| <a name="output_proxy_target_endpoint"></a> [proxy_target_endpoint](#output_proxy_target_endpoint)                               | Hostname for the target RDS DB Instance. Only returned for `RDS_INSTANCE` type                                                                                           |
| <a name="output_proxy_target_id"></a> [proxy_target_id](#output_proxy_target_id)                                                 | Identifier of `db_proxy_name`, `target_group_name`, `target type` (e.g. `RDS_INSTANCE` or `TRACKED_CLUSTER`), and resource identifier separated by forward slashes (`/`) |
| <a name="output_proxy_target_port"></a> [proxy_target_port](#output_proxy_target_port)                                           | Port for the target RDS DB instance or Aurora DB cluster                                                                                                                 |
| <a name="output_proxy_target_rds_resource_id"></a> [proxy_target_rds_resource_id](#output_proxy_target_rds_resource_id)          | Identifier representing the DB instance or DB cluster target                                                                                                             |
| <a name="output_proxy_target_target_arn"></a> [proxy_target_target_arn](#output_proxy_target_target_arn)                         | Amazon Resource Name (ARN) for the DB instance or DB cluster                                                                                                             |
| <a name="output_proxy_target_tracked_cluster_id"></a> [proxy_target_tracked_cluster_id](#output_proxy_target_tracked_cluster_id) | DB Cluster identifier for the DB instance target. Not returned unless manually importing an `RDS_INSTANCE` target that is part of a DB cluster                           |
| <a name="output_proxy_target_type"></a> [proxy_target_type](#output_proxy_target_type)                                           | Type of target. e.g. `RDS_INSTANCE` or `TRACKED_CLUSTER`                                                                                                                 |

<!-- markdownlint-restore -->

## Share the Love

Like this project? Please give it a ★ on [our GitHub](https://github.com/cloudposse/terraform-aws-rds-db-proxy)! (it helps us **a lot**)

Are you using this project or any of our other projects? Consider [leaving a testimonial][testimonial]. =)

## Related Projects

Check out these related projects.

- [terraform-aws-rds-cluster](https://github.com/cloudposse/terraform-aws-rds-cluster) - Terraform module to provision an RDS Aurora cluster for MySQL or Postgres.
- [terraform-aws-rds](https://github.com/cloudposse/terraform-aws-rds) - Terraform module to provision AWS RDS instances.
- [terraform-aws-rds-cloudwatch-sns-alarms](https://github.com/cloudposse/terraform-aws-rds-cloudwatch-sns-alarms) - Terraform module that configures important RDS alerts using CloudWatch and sends them to an SNS topic.
- [terraform-aws-rds-replica](https://github.com/cloudposse/terraform-aws-rds-replica) - Terraform module to provision AWS RDS replica instances. These are best suited for reporting purposes.
- [terraform-aws-backup](https://github.com/cloudposse/terraform-aws-backup) - Terraform module to provision AWS Backup, a fully managed backup service that makes it easy to centralize and automate the back up of data across AWS services such as Amazon EBS volumes, Amazon EC2 instances, Amazon RDS databases, Amazon DynamoDB tables, Amazon EFS file systems, and AWS Storage Gateway volumes.

## Help

**Got a question?** We got answers.

File a GitHub [issue](https://github.com/cloudposse/terraform-aws-rds-db-proxy/issues), send us an [email][email] or join our [Slack Community][slack].

[![README Commercial Support][readme_commercial_support_img]][readme_commercial_support_link]

## DevOps Accelerator for Startups

We are a [**DevOps Accelerator**][commercial_support]. We'll help you build your cloud infrastructure from the ground up so you can own it. Then we'll show you how to operate it and stick around for as long as you need us.

[![Learn More](https://img.shields.io/badge/learn%20more-success.svg?style=for-the-badge)][commercial_support]

Work directly with our team of DevOps experts via email, slack, and video conferencing.

We deliver 10x the value for a fraction of the cost of a full-time engineer. Our track record is not even funny. If you want things done right and you need it done FAST, then we're your best bet.

- **Reference Architecture.** You'll get everything you need from the ground up built using 100% infrastructure as code.
- **Release Engineering.** You'll have end-to-end CI/CD with unlimited staging environments.
- **Site Reliability Engineering.** You'll have total visibility into your apps and microservices.
- **Security Baseline.** You'll have built-in governance with accountability and audit logs for all changes.
- **GitOps.** You'll be able to operate your infrastructure via Pull Requests.
- **Training.** You'll receive hands-on training so your team can operate what we build.
- **Questions.** You'll have a direct line of communication between our teams via a Shared Slack channel.
- **Troubleshooting.** You'll get help to triage when things aren't working.
- **Code Reviews.** You'll receive constructive feedback on Pull Requests.
- **Bug Fixes.** We'll rapidly work with you to fix any bugs in our projects.

## Slack Community

Join our [Open Source Community][slack] on Slack. It's **FREE** for everyone! Our "SweetOps" community is where you get to talk with others who share a similar vision for how to rollout and manage infrastructure. This is the best place to talk shop, ask questions, solicit feedback, and work together as a community to build totally _sweet_ infrastructure.

## Discourse Forums

Participate in our [Discourse Forums][discourse]. Here you'll find answers to commonly asked questions. Most questions will be related to the enormous number of projects we support on our GitHub. Come here to collaborate on answers, find solutions, and get ideas about the products and services we value. It only takes a minute to get started! Just sign in with SSO using your GitHub account.

## Newsletter

Sign up for [our newsletter][newsletter] that covers everything on our technology radar. Receive updates on what we're up to on GitHub as well as awesome new projects we discover.

## Office Hours

[Join us every Wednesday via Zoom][office_hours] for our weekly "Lunch & Learn" sessions. It's **FREE** for everyone!

[![zoom](https://img.cloudposse.com/fit-in/200x200/https://cloudposse.com/wp-content/uploads/2019/08/Powered-by-Zoom.png")][office_hours]

## Contributing

### Bug Reports & Feature Requests

Please use the [issue tracker](https://github.com/cloudposse/terraform-aws-rds-db-proxy/issues) to report any bugs or file feature requests.

### Developing

If you are interested in being a contributor and want to get involved in developing this project or [help out](https://cpco.io/help-out) with our other projects, we would love to hear from you! Shoot us an [email][email].

In general, PRs are welcome. We follow the typical "fork-and-pull" Git workflow.

1.  **Fork** the repo on GitHub
2.  **Clone** the project to your own machine
3.  **Commit** changes to your own branch
4.  **Push** your work back up to your fork
5.  Submit a **Pull Request** so that we can review your changes

**NOTE:** Be sure to merge the latest changes from "upstream" before making a pull request!

## Copyright

Copyright © 2017-2023 [Cloud Posse, LLC](https://cpco.io/copyright)

## License

[![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](https://opensource.org/licenses/Apache-2.0)

See [LICENSE](LICENSE) for full details.

```text
Licensed to the Apache Software Foundation (ASF) under one
or more contributor license agreements.  See the NOTICE file
distributed with this work for additional information
regarding copyright ownership.  The ASF licenses this file
to you under the Apache License, Version 2.0 (the
"License"); you may not use this file except in compliance
with the License.  You may obtain a copy of the License at

  https://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing,
software distributed under the License is distributed on an
"AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
KIND, either express or implied.  See the License for the
specific language governing permissions and limitations
under the License.
```

## Trademarks

All other trademarks referenced herein are the property of their respective owners.

## About

This project is maintained and funded by [Cloud Posse, LLC][website]. Like it? Please let us know by [leaving a testimonial][testimonial]!

[![Cloud Posse][logo]][website]

We're a [DevOps Professional Services][hire] company based in Los Angeles, CA. We ❤️ [Open Source Software][we_love_open_source].

We offer [paid support][commercial_support] on all of our projects.

Check out [our other projects][github], [follow us on twitter][twitter], [apply for a job][jobs], or [hire us][hire] to help with your cloud strategy and implementation.

### Contributors

<!-- markdownlint-disable -->

| [![Erik Osterman][osterman_avatar]][osterman_homepage]<br/>[Erik Osterman][osterman_homepage] | [![Andriy Knysh][aknysh_avatar]][aknysh_homepage]<br/>[Andriy Knysh][aknysh_homepage] | [![RB][nitrocode_avatar]][nitrocode_homepage]<br/>[RB][nitrocode_homepage] |
| --------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------- | -------------------------------------------------------------------------- |

<!-- markdownlint-restore -->

[osterman_homepage]: https://github.com/osterman
[osterman_avatar]: https://img.cloudposse.com/150x150/https://github.com/osterman.png
[aknysh_homepage]: https://github.com/aknysh
[aknysh_avatar]: https://img.cloudposse.com/150x150/https://github.com/aknysh.png
[nitrocode_homepage]: https://github.com/nitrocode
[nitrocode_avatar]: https://img.cloudposse.com/150x150/https://github.com/nitrocode.png

[![README Footer][readme_footer_img]][readme_footer_link]
[![Beacon][beacon]][website]

<!-- markdownlint-disable -->

[logo]: https://cloudposse.com/logo-300x69.svg
[docs]: https://cpco.io/docs?utm_source=github&utm_medium=readme&utm_campaign=cloudposse/terraform-aws-rds-db-proxy&utm_content=docs
[website]: https://cpco.io/homepage?utm_source=github&utm_medium=readme&utm_campaign=cloudposse/terraform-aws-rds-db-proxy&utm_content=website
[github]: https://cpco.io/github?utm_source=github&utm_medium=readme&utm_campaign=cloudposse/terraform-aws-rds-db-proxy&utm_content=github
[jobs]: https://cpco.io/jobs?utm_source=github&utm_medium=readme&utm_campaign=cloudposse/terraform-aws-rds-db-proxy&utm_content=jobs
[hire]: https://cpco.io/hire?utm_source=github&utm_medium=readme&utm_campaign=cloudposse/terraform-aws-rds-db-proxy&utm_content=hire
[slack]: https://cpco.io/slack?utm_source=github&utm_medium=readme&utm_campaign=cloudposse/terraform-aws-rds-db-proxy&utm_content=slack
[linkedin]: https://cpco.io/linkedin?utm_source=github&utm_medium=readme&utm_campaign=cloudposse/terraform-aws-rds-db-proxy&utm_content=linkedin
[twitter]: https://cpco.io/twitter?utm_source=github&utm_medium=readme&utm_campaign=cloudposse/terraform-aws-rds-db-proxy&utm_content=twitter
[testimonial]: https://cpco.io/leave-testimonial?utm_source=github&utm_medium=readme&utm_campaign=cloudposse/terraform-aws-rds-db-proxy&utm_content=testimonial
[office_hours]: https://cloudposse.com/office-hours?utm_source=github&utm_medium=readme&utm_campaign=cloudposse/terraform-aws-rds-db-proxy&utm_content=office_hours
[newsletter]: https://cpco.io/newsletter?utm_source=github&utm_medium=readme&utm_campaign=cloudposse/terraform-aws-rds-db-proxy&utm_content=newsletter
[discourse]: https://ask.sweetops.com/?utm_source=github&utm_medium=readme&utm_campaign=cloudposse/terraform-aws-rds-db-proxy&utm_content=discourse
[email]: https://cpco.io/email?utm_source=github&utm_medium=readme&utm_campaign=cloudposse/terraform-aws-rds-db-proxy&utm_content=email
[commercial_support]: https://cpco.io/commercial-support?utm_source=github&utm_medium=readme&utm_campaign=cloudposse/terraform-aws-rds-db-proxy&utm_content=commercial_support
[we_love_open_source]: https://cpco.io/we-love-open-source?utm_source=github&utm_medium=readme&utm_campaign=cloudposse/terraform-aws-rds-db-proxy&utm_content=we_love_open_source
[terraform_modules]: https://cpco.io/terraform-modules?utm_source=github&utm_medium=readme&utm_campaign=cloudposse/terraform-aws-rds-db-proxy&utm_content=terraform_modules
[readme_header_img]: https://cloudposse.com/readme/header/img
[readme_header_link]: https://cloudposse.com/readme/header/link?utm_source=github&utm_medium=readme&utm_campaign=cloudposse/terraform-aws-rds-db-proxy&utm_content=readme_header_link
[readme_footer_img]: https://cloudposse.com/readme/footer/img
[readme_footer_link]: https://cloudposse.com/readme/footer/link?utm_source=github&utm_medium=readme&utm_campaign=cloudposse/terraform-aws-rds-db-proxy&utm_content=readme_footer_link
[readme_commercial_support_img]: https://cloudposse.com/readme/commercial-support/img
[readme_commercial_support_link]: https://cloudposse.com/readme/commercial-support/link?utm_source=github&utm_medium=readme&utm_campaign=cloudposse/terraform-aws-rds-db-proxy&utm_content=readme_commercial_support_link
[share_twitter]: https://twitter.com/intent/tweet/?text=terraform-aws-rds-db-proxy&url=https://github.com/cloudposse/terraform-aws-rds-db-proxy
[share_linkedin]: https://www.linkedin.com/shareArticle?mini=true&title=terraform-aws-rds-db-proxy&url=https://github.com/cloudposse/terraform-aws-rds-db-proxy
[share_reddit]: https://reddit.com/submit/?url=https://github.com/cloudposse/terraform-aws-rds-db-proxy
[share_facebook]: https://facebook.com/sharer/sharer.php?u=https://github.com/cloudposse/terraform-aws-rds-db-proxy
[share_googleplus]: https://plus.google.com/share?url=https://github.com/cloudposse/terraform-aws-rds-db-proxy
[share_email]: mailto:?subject=terraform-aws-rds-db-proxy&body=https://github.com/cloudposse/terraform-aws-rds-db-proxy
[beacon]: https://ga-beacon.cloudposse.com/UA-76589703-4/cloudposse/terraform-aws-rds-db-proxy?pixel&cs=github&cm=readme&an=terraform-aws-rds-db-proxy

<!-- markdownlint-restore -->
