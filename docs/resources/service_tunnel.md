---
page_title: "banyan_service_tunnel Resource - terraform-provider-banyan"
subcategory: ""
description: |-
  Resource used for lifecycle management of service tunnels. In order to properly function this resource must be utilized with the banyanaccesstier resource or banyanaccesstier2 Terraform registry modules. Please see the example below and in the Terraform modules for the respective cloud provider. For more information on service tunnels see the documentation https://docs.banyansecurity.io/docs/feature-guides/service-tunnels/
---

# banyan_service_tunnel (Resource)

Resource used for lifecycle management of service tunnels. In order to properly function this resource must be utilized with the banyan_accesstier resource or banyan_accesstier2 Terraform registry modules. Please see the example below and in the Terraform modules for the respective cloud provider. For more information on service tunnels see the documentation https://docs.banyansecurity.io/docs/feature-guides/service-tunnels/

## Example Usage
```terraform
resource "banyan_api_key" "example" {
  name        = "example api key"
  description = "example api key"
  scope       = "access_tier"
}

resource "banyan_accesstier" "example" {
  name         = "example"
  address      = "*.example.mycompany.com"
  api_key_id   = banyan_api_key.example.name
  tunnel_cidrs = ["10.10.1.0/24"]
}

resource "banyan_service_tunnel" "example" {
  name         = "example-anyone-high"
  description  = "tunnel allowing anyone with a high trust level"
  access_tiers = [banyan_accesstier.example.name]
  policy       = banyan_policy_tunnel.anyone-high.id
}

resource "banyan_policy_tunnel" "anyone-high" {
  name        = "allow anyone"
  description = "${banyan_accesstier.example.name} allow"
  access {
    roles       = ["ANY"]
    trust_level = "High"
  }
}
```

## Example Service Tunnel with L4 Policy
```terraform
resource "banyan_api_key" "example" {
  name        = "example api key"
  description = "example api key"
  scope       = "access_tier"
}

resource "banyan_accesstier" "example" {
  name         = "example"
  address      = "*.example.mycompany.com"
  api_key_id   = banyan_api_key.example.name
  tunnel_cidrs = ["10.10.0.0/16"]
}

resource "banyan_service_tunnel" "users" {
  name         = "corporate network"
  description  = "tunnel allowing anyone with a high trust level access to 443"
  access_tiers = [banyan_accesstier.example.name]
  policy       = banyan_policy_tunnel.anyone-high.id
}

resource "banyan_service_tunnel" "administrators" {
  name         = "corporate network admin"
  description  = "tunnel allowing administrators access to the networks"
  access_tiers = [banyan_accesstier.example.name]
  policy       = banyan_policy_tunnel.administrators.id
}

resource "banyan_policy_tunnel" "anyone-high" {
  name        = "corporate-network-users"
  description = "${banyan_accesstier.example.name} allow users"
  access {
    roles       = ["Everyone"]
    trust_level = "High"
    l4_access {
      allow {
        cidrs     = ["10.10.10.0/24"]
        protocols = ["TCP"]
        ports     = ["443"]
      }
    }
  }
}

resource "banyan_policy_tunnel" "administrators" {
  name        = "corporate-network-admin"
  description = "${banyan_accesstier.example.name} allow only administrators access to the entire network"
  access {
    roles       = ["Everyone"]
    trust_level = "High"
    l4_access {
      allow {
        cidrs     = ["10.10.10.0/24"]
        protocols = ["TCP"]
        ports     = ["443"]
      }
    }
  }
}
```
In this example an access tier is configured to tunnel `10.10.0.0/16`. A service tunnel is configured to utilize this access tier, and a policy is attached which only allows users with a `High` trust level access to services running on port 443 in the subnet `10.10.1.0/24`. An additional service tunnel and policy allows administrators access to the entire network behind the tunnel.

<!-- schema generated by tfplugindocs -->
## Schema

### Required

- `name` (String) Name of the service tunnel
- `policy` (String) Policy ID to be attached to this service tunnel

### Optional

- `access_tiers` (Set of String) Names of the access_tiers which the service tunnel should be associated with
- `cluster` (String, Deprecated) (Depreciated) Sets the cluster / shield for the service
- `connectors` (Set of String) Names of the connectors which the service tunnel should be associated with
- `description` (String) Description of the service tunnel
- `description_link` (String) Link shown to the end user of the banyan app for this service
- `public_cidrs_exclude` (Set of String) Specifies public IP addresses in CIDR notation that should be excluded from the tunnel, ex: 8.8.12.0/24.
- `public_cidrs_include` (Set of String) Specifies public IP addresses in CIDR notation that should be included in the tunnel, ex: 8.8.0.0/16.
- `public_domains_exclude` (Set of String) Specifies the domains that should be that should be excluded from the tunnel, ex: zoom.us
- `public_domains_include` (Set of String) Specifies the domains that should be that should be included in the tunnel, ex: cnn.com

### Read-Only

- `id` (String) ID of the service tunnel key in Banyan
