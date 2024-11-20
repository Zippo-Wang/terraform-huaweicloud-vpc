# The Terraform module of HUAWEI Cloud VPC service

What capabilities does the current Module provide:

+ One-click deployment of VPC and related resources.
+ Use the resource's name to query the resource's ID by data-sources.

## Usage

### Create a VPC and two subnets, no security group

```hcl
module "vpc_service" {
  source = "terraform-huaweicloud-modules/vpc-service"

  vpc_name       = "module-single-vpc"
  vpc_cidr_block = "172.16.0.0/16"

  subnet_configuration = [
    {
      name="module-single-master-subnet",
      cidr="172.16.66.0/24"
    },
    {
      name="module-single-standby-subnet",
      cidr="172.16.86.0/24"
    },
  ]

  is_security_group_create = false
}
```

### Create a security group and three security group rules (contains a self access rule in security group)

```hcl
module "vpc_service" {
  source = "terraform-huaweicloud-modules/vpc-service"

  is_vpc_create = false

  is_security_group_create   = true
  security_group_name        = "module-single-security-group"
  security_group_description = "Created by terraform module"

  subnet_configuration = [
    {
      description="Created by terraform module",
      direction="ingress",
      ethertype="IPv6",
      protocol="tcp",
      ports="22",
      remote_ip_prefix="::/0",
      action="deny",
      priority=100,
    },
  ]
}
```

### Query resource IDs using resource names

```hcl
module "vpc_service" {
  source = "terraform-huaweicloud-modules/vpc-service"

  query_vpc_names            = ["module-single-vpc"]
  query_subnet_names         = ["module-single-master-subnet", "module-single-standby-subnet"]
  query_security_group_names = ["module-single-security-group"]
}
```

## Contributing

Report issues/questions/feature requests in the [issues](https://github.com/terraform-huaweicloud-modules/terraform-huaweicloud-vpc/issues/new)
section.

Full contributing [guidelines are covered here](.github/how_to_contribute.md).

## Requirements

| Name | Version |
|------|---------|
| Terraform | >= 1.3.0 |
| Huaweicloud Provider | >= 1.40.0 |

## Resources

| Name | Type |
|------|------|
| huaweicloud_vpc.this | resource |
| huaweicloud_vpc_subnet.this | resource |
| huaweicloud_networking_secgroup.this | resource |
| huaweicloud_networking_secgroup_rule.in_v4_self_group | resource |
| huaweicloud_networking_secgroup_rule.this | resource |
| huaweicloud_vpc_address_group.security_group_rules_auto_created | resource |
| huaweicloud_networking_secgroup_rule.remote_address_group | resource |
| data.huaweicloud_vpcs.this | data-source |
| data.huaweicloud_vpc_subnets.this | data-source |
| data.huaweicloud_networking_secgroups.this | data-source |
| data.huaweicloud_networking_secgroup_rules.this | data-source |

## Inputs

<!-- markdownlint-disable MD013 -->
| Name | Description | Type | Default | Required |
|------|-------------|------|---------|----------|
| enterprise_project_id | Used to specify whether the resource is created under the enterprise project (this parameter is only valid for enterprise users) | string | "" | N |
| availability_zone | The availability zone to which the VPC subnet resource belongs | string | "" | N |
| is_vpc_create | Controls whether a VPC should be created (it affects all VPC related resources under this module) | bool | true | N |
| vpc_name | The name of the VPC resource | string | "" | Y (Unless is_vpc_create is specified as false) |
| vpc_cidr | The CIDR block of the VPC resource | string | "192.168.0.0/16" | N |
| vpc_description | The description of the VPC resource | string | "" | N |
| vpc_secondary_cidrs | The secondary CIDR blocks of the VPC resource | list(string) | <pre>[]</pre> | N |
| vpc_tags | The key/value pairs to associte with the VPC resource | map(string) | <pre>{}</pre> | N |
| subnets_configuration | The configuration for the subnet resources to which the VPC belongs | <pre>list(object({<br>  name           = string<br>  cidr           = string<br>  description    = optional(string, "")<br>  ipv6_enabled   = optional(bool, null)<br>  dhcp_enabled   = optional(bool, null)<br>  dns_list       = optional(list(string), [])<br>  tags           = optional(map(string), {})<br>}))</pre> | <pre>[]</pre> | N |
| is_security_group_create | Controls whether a security group should be created (it affects all security group related resources under this module) | bool | true | N |
| security_group_name | The name of the security group resource" | string | "" | Y (Unless is_security_group_create is specified as false) |
| security_group_description | The description of the security group resource | string | "" | N |
| security_group_rules_configuration | The configuration for security group rule resources to which the security group belongs<br>Notes:<br>1. The usage priority of the parameters remote_ip_prefix, remote_group_id, and remote_address_group_id is: remote_group_id > remote_address_group_id > remote_ip_prefix<br>2. The parameters remote_address_group_id and remote_addresses cannot be configured at the same time<br>3. The parameters address_group_name is required if remote_addresses is configured | <pre>list(object({<br>  description             = optional(string, "")<br>  direction               = optional(string, "ingress")<br>  ethertype               = optional(string, "IPv4")<br>  protocol                = optional(string, "")<br>  ports                   = optional(string, "")<br>  remote_ip_prefix        = optional(string, "0.0.0.0/0")<br>  remote_group_id         = optional(string, "")<br>  remote_address_group_id = optional(string, "")<br>  address_group_name      = optional(string, "")<br>  remote_addresses        = optional(list(string), [])<br>  action                  = optional(string, "allow")<br>  priority                = optional(number, 0)<br>}))</pre> | <pre>[]</pre> | N |
| query_vpc_names | The VPC name list used to query the resource IDs | list(string) | <pre>[]</pre> | N |
| query_subnet_names | The subnet name list used to query the resource IDs | list(string) | <pre>[]</pre> | N |
| query_security_group_names | The security group name list used to query the resource IDs | list(string) | <pre>[]</pre> | N |
<!-- markdownlint-enable MD013 -->

## Outputs

| Name | Description |
|------|-------------|
| vpc_id | The ID of the VPC resource |
| vpc_cidr | The CIDR block of the VPC resource |
| subnet_cidrs | The CIDR list of the subnet resources to which the VPC resource belongs |
| subnet_ids | The ID list of the subnet resources to which the VPC resource belongs |
| security_group_id | The ID of the security group resource |
| security_group_rules | All rules to which the security group resource belongs |
| queried_vpc_ids | The ID list of the VPC resources for data-source query by resource name |
| queried_subnet_ids | The ID list of the subnet resources for data-source query by resource name |
| queried_security_group_ids | The ID list of the security group resources for data-source query by resource name |
