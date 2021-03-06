# Azure Alerting
[![Changelog](https://img.shields.io/badge/changelog-release-green.svg)](CHANGELOG.md) [![Notice](https://img.shields.io/badge/notice-copyright-yellow.svg)](NOTICE) [![Apache V2 License](https://img.shields.io/badge/license-Apache%20V2-orange.svg)](LICENSE) [![TF Registry](https://img.shields.io/badge/terraform-registry-blue.svg)](https://registry.terraform.io/modules/claranet/alerting/azurerm/)

Azure module to create some [Azure Monitor Alerts](https://docs.microsoft.com/en-us/azure/azure-monitor/platform/alerts-overview)
with an [Action Group](https://docs.microsoft.com/en-us/azure/azure-monitor/platform/action-groups) for notifications destination.
This module handles alerts of type:
  - [Activity Log alerts](https://docs.microsoft.com/en-us/azure/azure-monitor/platform/alerts-activity-log)

## Version compatibility

| Module version    | Terraform version | AzureRM version |
|-------------------|-------------------|-----------------|
| >= 3.x.x          | 0.12.x            | >= 2.0          |
| >= 2.x.x, < 3.x.x | 0.12.x            | <  2.0          |
| <  2.x.x          | 0.11.x            | <  2.0          |

## Usage

This module is optimized to work with the [Claranet terraform-wrapper](https://github.com/claranet/terraform-wrapper) tool
which set some terraform variables in the environment needed by this module.
More details about variables set by the `terraform-wrapper` available in the [documentation](https://github.com/claranet/terraform-wrapper#environment).

```hcl
module "azure-region" {
  source  = "claranet/regions/azurerm"
  version = "x.x.x"

  azure_region = var.azure_region
}

module "rg" {
  source  = "claranet/rg/azurerm"
  version = "x.x.x"

  location    = module.azure-region.location
  client_name = var.client_name
  environment = var.environment
  stack       = var.stack
}

module "alerting" {
  source  = "claranet/alerting/azurerm"
  version = "x.x.x"

  location_short = module.azure-region.location_short
  client_name    = var.client_name
  environment    = var.environment
  stack          = var.stack

  resource_group_name     = module.rg.resource_group_name
  action_group_short_name = "Alerting"

  action_group_webhooks = {
    PagerDuty = "https://events.pagerduty.com/integration/{integration-UID}/enqueue"
    Slack     = "https://hooks.slack.com/services/{azerty}/XXXXXXXXXXXXXXx/{hook-key}"
  }

  activity_log_alerts = {
    "service-health" = {
      description         = "ServiceHealth global Subscription alerts"
      resource_group_name = module.rg.resource_group_name
      scopes              = [format("/subscriptions/%s", var.azure_subscription_id)]
      criteria = {
        category = "ServiceHealth"
      }
    }

    "security-center" = {
      custom_name         = "${var.stack}-global-security-center"
      description         = "Security Center global Subscription alerts"
      resource_group_name = module.rg.resource_group_name
      scopes              = [format("/subscriptions/%s", var.azure_subscription_id)]
      criteria = {
        category = "Security"
        level    = "Error"
      }
    }
  }

  extra_tags = {
    purpose = "alerting testing"
  }
}

```

## Inputs

| Name | Description | Type | Default | Required |
|------|-------------|------|---------|:-----:|
| action\_group\_emails | Map of Emails to notify. Example: `{ ml-devops = devops@contoso.com }` | `map(string)` | `{}` | no |
| action\_group\_short\_name | Action Group short name | `string` | n/a | yes |
| action\_group\_webhooks | Map of Webhooks to notify. Example: `{ PagerDuty = 'https://events.pagerduty.com/integration/abcdefgh12345azerty/enqueue' }` | `map(string)` | `{}` | no |
| activity\_log\_alerts | Map of Activity log Alerts | `any` | `{}` | no |
| client\_name | Client name/account used in naming | `string` | n/a | yes |
| custom\_action\_group\_name | Optional custom Action Group name | `string` | `"null"` | no |
| environment | Project environment | `string` | n/a | yes |
| extra\_tags | Extra tags to set on each created resource. | `map(string)` | `{}` | no |
| location\_short | Short string for Azure location. | `string` | n/a | yes |
| name\_prefix | Optional prefix for resources names | `string` | `""` | no |
| resource\_group\_name | Resource group name | `string` | n/a | yes |
| stack | Project stack name | `string` | n/a | yes |

## Outputs

| Name | Description |
|------|-------------|
| action\_group\_id | Notification Action Group ID |
| action\_group\_name | Notification Action Group name |
| activity\_log\_alerts | Activity log alerts attributes |

## Related documentation

Terraform resource documentation: [terraform.io/docs/providers/azurerm/r/monitor_activity_log_alert.html](https://www.terraform.io/docs/providers/azurerm/r/monitor_activity_log_alert.html)

Microsoft Azure documentation: 
  - [Activity Log alerts](https://docs.microsoft.com/en-us/azure/azure-monitor/platform/alerts-activity-log)
