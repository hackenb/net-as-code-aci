[![Terraform Version](https://img.shields.io/badge/terraform-%5E1.8-blue)](https://www.terraform.io)

# Network-as-Code - Simple example for ACI

This example is part of the Cisco [*Network as Code*](https://netascode.cisco.com) project. Its goal is to allow users to instantiate network fabrics in minutes using an easy to use, opinionated data model. It takes away the complexity of having to deal with references, dependencies or loops. By completely separating data (defining variables) from logic (infrastructure declaration), it allows the user to focus on describing the intended configuration while using a set of maintained and tested Terraform Modules without the need to understand the low-level ACI object model. More information can be found here: [https://netascode.cisco.com/solutions/aci/simple_example](https://netascode.cisco.com/solutions/aci/simple_example).

## Current configuration

This repository contains a working ACI deployment example for a DEV tenant with access-policy and tenant-side networking components configured through Net-as-Code YAML and applied with OpenTofu.

### Access policies

The following access-policy objects are configured under `apic.access_policies`:

- VLAN pools:
  - `DEV-VLAN-POOL` with range `200-250`
  - `DEV-L3-VLAN-POOL` with range `1501-1501`
- Physical domain:
  - `DEV-PHYSICAL-DOMAIN` mapped to `DEV-VLAN-POOL`
- Routed domain:
  - `DEV-L3-DOMAIN` mapped to `DEV-L3-VLAN-POOL`
- AAEP:
  - `DEV-AAEP` attached to `DEV-PHYSICAL-DOMAIN` and `DEV-L3-DOMAIN`
- Leaf switch profiles:
  - `LEAF101`
  - `LEAF102`
  - `LEAF101-102`
- Interface profiles:
  - `LEAF101-INT-PROF`
  - `LEAF102-INT-PROF`
  - `LEAF101-102-INT-PROF`
- Interface policy groups:
  - `DEV-L3-IF-PG` using `DEV-AAEP` and built-in `system-cdp-disabled`
  - `DEV-SERVER-VPC-PG` using `DEV-AAEP`, `system-lldp-enabled`, and `system-lacp-active`

### Tenant configuration

The DEV tenant includes:

- VRF: `DEV-VRF`
- Bridge domains:
  - `10.1.200.0_24`
  - `10.1.201.0_24`
  - `10.1.202.0_24`
- Application profile: `VLANS`
- Endpoint groups:
  - `VLAN200`
  - `VLAN201`
  - `VLAN202`

Each EPG is assigned to `DEV-PHYSICAL-DOMAIN` and uses a static VPC trunk uplink on nodes 101/102 with encapsulation matching the VLAN number.

### Contracts

The example defines the following contracts:

- `WEB-CONTRACT`
  - `VLAN200` consumes it
  - `VLAN201` provides it
- `MYSQL-3306-CONTRACT`
  - `VLAN201` consumes it
  - `VLAN202` provides it

### Files

- `main.tf` - OpenTofu provider/module configuration and management flags
- `data/fabric_access_policies.nac.yaml` - access-policy model
- `data/tenant_DEV.nac.yaml` - DEV tenant, bridge domains, EPGs, and contracts

### Apply

Run the following from the repo root:

```bash
terraform init
terraform apply -auto-approve
```

This configuration is intended as a working example for ACI access and tenant policy modeling with Cisco Net-as-Code and Terraform.
