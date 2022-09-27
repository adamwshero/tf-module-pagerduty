[![SWUbanner](https://raw.githubusercontent.com/vshymanskyy/StandWithUkraine/main/banner2-direct.svg)](https://github.com/vshymanskyy/StandWithUkraine/blob/main/docs/README.md)

![Terraform](https://cloudarmy.io/tldr/images/tf_aws.jpg)
<br>
<br>
<br>
<br>
![GitHub tag (latest by date)](https://img.shields.io/github/v/tag/adamwshero/terraform-pagerduty-service?color=lightgreen&label=latest%20tag%3A&style=for-the-badge)
<br>
<br>
# terraform-pagerduty-service

Through its SaaS-based platform, PagerDuty empowers developers, DevOps, IT operations and business leaders to prevent and resolve business-impacting incidents for exceptional customer experience. When revenue and brand reputation depends on customer satisfaction, PagerDuty arms organizations with the insight to proactively manage events that may impact customers across their IT environment. With hundreds of native integrations, on-call scheduling and escalations, machine learning, business-wide response orchestration, analytics, and much more, PagerDuty gets the right data in the hands of the right people in real time, every time.
<br>

## Module Capabilities
- Create a Pagerduty Service
- Supports many maintenance windows
- Optional Incident Urgency Rule
- Optional Scheduled Actions
- Optional SNS Topic for integration notifications
- Ability to Subscribe the Pagerduty service to an SNS topic (if enabled)
- Optional Slack extension to a specified Slack channel
- Option to create a Service integration (e.g. CloudWatch, DataDog, etc.)

[Pagerduty Service](https://support.pagerduty.com/docs/services-and-integrations) represents something you monitor (like a web service, email service, or database service). It is a container for related incidents that associates them with escalation policies.

## Examples
Look at our complete [Terraform examples](latest/examples/terraform/) where you can get a better context of usage. The Terragrunt example can be viewed directly from GitHub.
<br>

## Assumptions
  * You already have access to a PagerDuty API key to create the service.
  * You have access to Slack to acquire the team, bot, and other information need for the extension to work.
<br>

## Usage
You can create a PagerDuty service that comes with its own SNS topic and a subcription to that topic. The module also creates the CloudWatch integration for you as well as a Slack extension for notifications to blast to your channel of choice.
<br>

### Terraform Example
```
module "pagerduty-service" {
  source = "git@github.com:adamwshero/terraform-pagerduty-service.git//.?ref=1.1.0"

  // PagerDuty Service
  name              = "My Critical Service"
  description       = "Service for all prod services."
  escalation_policy = "My Escalation Policy Name"
  alert_creation    = "create_alerts_and_incidents"
  resolve_timeout   = 14400
  ack_timeout       = 600
  token             = file(./my_pagerduty_api_key.yaml)

  incident_urgency_rule = [{
    type    = "constant"
    urgency = "low"

    during_support_hours = [{
      type    = "constant"
      urgency = "low"
    }]

    outside_support_hours = [{
      type    = "constant"
      urgency = "low"
    }]
  }]

  // SNS Topic
  prefix       = "my-prefix"
  service_name = "AcmeCorp-Elasticsearch"

  // Slack Extension
  create_slack_extension = true

  extension_name     = "DevOps: Slack"
  schema_webhook     = "Generic V1 Webhook"
  config = templatefile("${path.module}/slack/config.json.tpl", {
    app_id             = "A1AAAAAAA"
    authed_user        = "A11AAA11AAA"
    bot_user_id        = "A111AAAA11A"
    slack_team_id      = "AAAAAA11A"
    slack_team_name    = "AcmeCorp"
    slack_channel      = "#devops-pagerduty"
    slack_channel_id   = "A11AA1AAA1A"
    configuration_url  = "https://acme-corp.slack.com/services/A111AAAAAAAA"
    referer            = "https://acmecorp.pagerduty.com/services/A1AAAA1/integrations?service_profile=1"
    notify_resolve     = true
    notify_trigger     = true
    notify_escalate    = true
    notify_acknowledge = true
    notify_assignments = true
    notify_annotate    = true
    high_urgency       = true
    low_urgency        = true
  })
```

### Terragrunt Example
```
locals {
  external_deps = read_terragrunt_config(find_in_parent_folders("external-deps.hcl"))
  account_vars  = read_terragrunt_config(find_in_parent_folders("account.hcl"))
  product_vars  = read_terragrunt_config(find_in_parent_folders("product.hcl"))
  env_vars      = read_terragrunt_config(find_in_parent_folders("env.hcl"))
  product       = local.product_vars.locals.product_name
  prefix        = local.product_vars.locals.prefix
  account       = local.account_vars.locals.account_id
  env           = local.env_vars.locals.env
  pagerduty_key = yamldecode(sops_decrypt_file("${get_terragrunt_dir()}/sops/api-key.sops.yaml"))
  tags = merge(
    local.env_vars.locals.tags,
    local.additional_tags
  )

  # Customize if needed
  additional_tags = {

  }
}

include {
  path = find_in_parent_folders()
}

terraform {
  source = "git@github.com:adamwshero/terraform-pagerduty-service.git//.?ref=1.1.0"
}

inputs = {
  // PagerDuty Service
  name              = "My Critical Service"
  description       = "Service for all prod services."
  escalation_policy = "My Escalation Policy Name"
  alert_creation    = "create_alerts_and_incidents"
  resolve_timeout   = 14400
  ack_timeout       = 600
  token             = local.pagerduty_key.key

  incident_urgency_rule = [{
    type    = "constant"
    urgency = "low"

    during_support_hours = [{
      type    = "constant"
      urgency = "low"
    }]

    outside_support_hours = [{
      type    = "constant"
      urgency = "low"
    }]
  }]

  // SNS Topic
  prefix       = "my-prefix"
  service_name = "AcmeCorp-Elasticsearch"

  // Slack Extension
  create_slack_extension = true

  extension_name     = "DevOps: Slack"
  schema_webhook     = "Generic V1 Webhook"
  config = templatefile("${get_terragrunt_dir()}/slack/config.json.tpl", {
    app_id             = "A1AAAAAAA"
    authed_user        = "A11AAA11AAA"
    bot_user_id        = "A111AAAA11A"
    slack_team_id      = "AAAAAA11A"
    slack_team_name    = "AcmeCorp"
    slack_channel      = "#devops-pagerduty"
    slack_channel_id   = "A11AA1AAA1A"
    configuration_url  = "https://acme-corp.slack.com/services/A111AAAAAAAA"
    referer            = "https://acmecorp.pagerduty.com/services/A1AAAA1/integrations?service_profile=1"
    notify_resolve     = true
    notify_trigger     = true
    notify_escalate    = true
    notify_acknowledge = true
    notify_assignments = true
    notify_annotate    = true
    high_urgency       = true
    low_urgency        = true
  })
}
```

<!-- BEGINNING OF PRE-COMMIT-TERRAFORM DOCS HOOK -->
## Requirements

| Name | Version |
|------|---------|
| <a name="requirement_aws"></a> [aws](#requirement\_aws) | >= 2.67.0 |
| <a name="requirement_terraform"></a> [terraform](#requirement\_terraform) | >= 0.14.0 
| <a name="requirement_terragrunt"></a> [terragrunt](#requirement\_terragrunt) | >= 0.28.0 |

## Providers

| Name | Version |
|------|---------|
| <a name="provider_aws"></a> [aws](#provider\_aws) | >= 2.67.0 |
| <a name="provider_pagerduty"></a> [pagerduty](#provider\_pagerduty) | >= 1.9.6 |


## Resources

| Name | Type |
|------|------|
| [pagerduty_service.rsm](https://registry.terraform.io/providers/PagerDuty/pagerduty/latest/docs/resources/service) | resource |
| [pagerduty_extension.rsm](https://registry.terraform.io/providers/PagerDuty/pagerduty/latest/docs/resources/extension) | resource |
| [pagerduty_service_integration.rsm](https://registry.terraform.io/providers/PagerDuty/pagerduty/latest/docs/data-sources/service_integration) | resource |
| [pagerduty_vendor.rsm](https://registry.terraform.io/providers/PagerDuty/pagerduty/latest/docs/data-sources/vendor) | resource |
| [aws_sns_topic.rsm](https://registry.terraform.io/providers/aaronfeng/aws/latest/docs/resources/sns_topic) | resource |
<br>

## Available Inputs

| Name                  | Resource            | Variable                | Data Type     | Default                         | Required? |
| --------------------- | --------------------|------------------------ | ------------- | ------------------------------- | ----------|
| Service Name          | pagerduty_service   | `name`                  | `string`      | `DevOps: Test Service`          | No        |
| Escalation Policy     | pagerduty_service   | `escalation_policy`     | `string`      | `""`                            | Yes       |
| Description           | pagerduty_service   | `description `          | `string`      | `""`                            | No        |
| Resolve Timeout       | pagerduty_service   | `resolve_timeout`       | `number`      | `14400`                         | No        | 
| Acknowledge Timeout   | pagerduty_service   | `ack_timeout`           | `number`      | `600`                           | No        |
| Alert Creation        | pagerduty_service   | `alert_creation`        | `string`      | `"create_alerts_and_incidents"` | Yes       |
| Escalation Policy     | pagerduty_service   | `escalation_policy`     | `string`      | `""`                            | Yes       |
| Incident Urgency Rule | pagerduty_service   | `incident_urgency_rule` | `set(object)` | `[]`                            | No        |
| Support Hours         | pagerduty_service   | `escalation_policy`     | `set(object)` | `[]`                            | No        |
| Scheduled Actions     | pagerduty_service   | `escalation_policy`     | `set(object)` | `[]`                            | No        |
| Prefix                | aws_sns_topic       | `prefix`                | `string`      | `""`                            | No        |
| Name                  | aws_sns_topic       | `name`                  | `string`      | `""`                            | Yes       |
| App Id                | pagerduty_extension | `app_id`                | `string`      | `""`                            | Yes       |
| Authorized User       | pagerduty_extension | `authed_user`           | `string`      | `""`                            | No        |
| Bot UserId            | pagerduty_extension | `bot_user_id`           | `string`      | `""`                            | Yes       |
| Channel               | pagerduty_extension | `slack_channel`         | `string`      | `""`                            | Yes       |
| Channel Id            | pagerduty_extension | `slack_channel_id`      | `string`      | `""`                            | Yes       |
| Configuration URL     | pagerduty_extension | `configuration_url`     | `string`      | `""`                            | Yes       | 
| Webhook URL           | pagerduty_extension | `url`                   | `string`      | `""`                            | Yes       |
| Notify on Resolve     | pagerduty_extension | `notify_resolve`        | `bool`        | `"true"`                        | No        |
| Notify on Trigger     | pagerduty_extension | `notify_trigger`        | `bool`        | `"true"`                        | No        |
| Notify on Escalate    | pagerduty_extension | `notify_escalate`       | `bool`        | `"true"`                        | No        |
| Notify on Acknowledge | pagerduty_extension | `notify_acknowledge`    | `bool`        | `"true"`                        | No        |
| Notify on Assignment  | pagerduty_extension | `notify_assignments`    | `bool`        | `"true"`                        | No        |
| Notify on Annotate    | pagerduty_extension | `notify_annotate`       | `bool`        | `"true"`                        | No        |
| Referer URL           | pagerduty_extension | `referer`               | `string`      | `""`                            | Yes       |
| Team Id               | pagerduty_extension | `slack_team_id`         | `string`      | `""`                            | Yes       |
| Team Name             | pagerduty_extension | `slack_team_name`       | `string`      | `""`                            | Yes       |
| Alert on High Urgency | pagerduty_extension | `high_urgency`          | `bool`        | `"true"`                        | No        |
| Alert on High Urgency | pagerduty_extension | `low_urgency`           | `bool`        | `"true"`                        | No        |

<br>

## Predetermined Inputs

| Name          | Resource                      | Variable      | Data Type | Default                                | Required? |
| --------------| ------------------------------|-------------- | --------- | -------------------------------------- | ----------|
| Name          | pagerduty_service_integration | `name`        | `string`  | `data.pagerduty_vendor.this.name`      | Yes       |
| Service       | pagerduty_service_integration | `service`     | `string`  | `pagerduty_service.this.id`            | Yes       |
| Vendor        | pagerduty_service_integration | `vendor`      | `string`  | `data.pagerduty_vendor.this.id`        | Yes       |
| Type          | pagerduty_service_integration | `type`        | `string`  | `"aws_cloudwatch_inbound_integration"` | Yes       |
| Vendor Name   | pagerduty_vendor              | `name`        | `string`  | `"CloudWatch"`                         | Yes       |

<br>

## Outputs

| Name                                     | Description                            |
|------------------------------------------|--------------------------------------- | 


## PagerDuty/Slack Extension Schema
 * https://developer.pagerduty.com/api-reference/YXBpOjExMjA5NTQ0-pager-duty-slack-integration-api (See /slack_schema.json)
 * https://developer.pagerduty.com/api-reference/b3A6Mjc0ODEzMg-list-extensions
<br>

## To-Do:
* Re-introduce alarm grouping options
* Create resource to handle runbook names and URL's.
* Create resource to handle service dependencies.
* Create CloudWatch metric alarm. (maybe)
* <s>Make module compatible with Terraform =>0.14.0</s>
<br>

## Known Issues
- PagerDuty is currently aware that the Slack extension must be manually authorized to connect to a given Slack channel once the service is created. There is no ETA on this fix.

