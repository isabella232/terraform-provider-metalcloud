---
layout: "metalcloud"
page_title: "Metalcloud: Getting started"
description: |-
  Getting started guide with the metalcloud provider.
---

# Metalcloud: Getting started

The Bigstep Metalcloud provider allows users to provision bare metal resources such as physical servers, switch condigurations, iSCSI drives etc.


## Provisioning a server

To provision a server:

```hcl
# Configure the metalcloud provider
provider "metalcloud" {
   user_email = var.user_email
   api_key = var.api_key 
   endpoint = var.endpoint
}

# Identity the ID of the volume template we want
data "metalcloud_volume_template" "centos76" {
  volume_template_label = "centos7-6"
}


resource "metalcloud_infrastructure" "my-infra102" {
  
  infrastructure_label = "my-terraform-infra102"
  datacenter_name = var.datacenter

  # Remove this to actually deploy changes, otherwise all changes will remain in edit mode only.
  prevent_deploy = true 
  
  instance_array {
    # Name of your cluster. Needs to obey DNS rules as it will translate into a DNS record.
    instance_array_label = "web-servers"

    instance_array_instance_count = 1
    instance_array_ram_gbytes = 8
    instance_array_processor_count = 1
    instance_array_processor_core_count = 8

    drive_array{
      drive_array_label = "web-servers-centos"
      drive_array_storage_type = "iscsi_hdd"

      # The size of the drive array in MBytes
      drive_size_mbytes_default = 49000

      # The id of the template we located earlier
      volume_template_id = tonumber(data.metalcloud_volume_template.centos76.id)
    }

    #one or more FW rules. By default all traffic is denied so we need at least one entry.
    firewall_rule {
      firewall_rule_description = "test fw rule"
      firewall_rule_port_range_start = 22
      firewall_rule_port_range_end = 22
      firewall_rule_source_ip_address_range_start="0.0.0.0"
      firewall_rule_source_ip_address_range_end="0.0.0.0"
      firewall_rule_protocol="tcp"
      firewall_rule_ip_address_type="ipv4"
    }
  }
}

```

## Authentication

Getting the API Key is typically done via the Bigstep Metal Cloud's API key section. Use it with a -var or as an env variable:

```bash
export TF_VAR_api_key="<yourkey>"
export TF_VAR_user="test@test.com"
export TF_VAR_endpoint="https://api.bigstep.com/metal-cloud"
export TF_VAR_datacenter="uk-reading"
terraform plan
```

!> Warning: Hard-coding credentials into any Terraform configuration is not recommended, and risks secret leakage should this file ever be committed to a public version control system. 

