# HWCloudTFSnippet

Archivo *.json con snippets para VS Code para llamados rápidos de elementos Terraform con Huawei Cloud

Descargar y almacenar en la ruta:

> `~USER_DATA\AppData\Roaming\Code\User\snippets\`

A continuación, se describe los snippets disponibles para Terraform utilizando Huawei Cloud.

## 1. Instanciar módulo HWCloudTFLib

- **Prefijo:** `tf-modHWCloudTFLib`

```hcl
module "HWCloudTFLib" {
    source = "git::https://github.com/JuanRoeder/HWCloudTFLib.git?ref=v1.6.1"
}

locals {
    enum_regions = module.HWCloudTFLib.enum_regions
    enum_commons = module.HWCloudTFLib.enum_commons
    enum_ecs = module.HWCloudTFLib.enum_ecs
    enum_evs = module.HWCloudTFLib.enum_evs
    enum_ims = module.HWCloudTFLib.enum_ims
    enum_vpc = module.HWCloudTFLib.enum_vpc
    enum_eip = module.HWCloudTFLib.enum_eip
    enum_vpn = module.HWCloudTFLib.enum_vpn
}
```

## 2. Instanciar variables.tf

- **Prefijo:** `tf-variables`

```hcl
variable "HWCLOUD_SK" {
    description = "Secret-key"
    type = string
    sensitive = true
}

variable "HWCLOUD_AK" {
    description = "Access-key"
    type = string
    sensitive = true
}

variable "HWCLOUD_REGION" {
    description = "Region"
    type = string
    default = ""
}

variable "HWCLOUD_PROJECTID" {
    description = "Project ID"
    type = string
    default = ""
}
```

## 3. Instanciar proveedor Huawei Cloud para Terraform

- **Prefijo:** `tf-required_providers`

```hcl
terraform {
    required_providers {
        huaweicloud = {
            source  = "huaweicloud/huaweicloud"
            version = ">= 1.36.0"
        }
    }
    required_version = ">= 0.13"
}
```

## 4. Instanciar credenciales del proveedor Huawei Cloud

- **Prefijo:** `tf-provider hwcloud`

```hcl
provider "huaweicloud" {
    region = "$region"
    access_key = "${var.HWCLOUD_AK}"
    secret_key = "${var.HWCLOUD_SK}"
}
```

## 5. Consulta data Enterprise Project

- **Prefijo:** `tf-data-eproject`

```hcl
data "huaweicloud_enterprise_project" "$enterprise_project" {
    name = "$name"
}
```

## 6. Consulta imagen Windows IMS

- **Prefijo:** `tf-data-image-windows`

```hcl
data "huaweicloud_images_image" "$windows" {
    most_recent  = true
    architecture = local.enum_ims.architecture.x86
    os           = local.enum_ims.os_type.windows.name
    visibility   = local.enum_ims.visibility.Public
    image_type   = local.enum_ims.image_type.ECS
    name         = local.enum_ims.os_type.windows.$Enum_Image_Name
}
```

## 7. Instanciar ECS resource

- **Prefijo:** `tf-ecs-resource`

```hcl
resource "huaweicloud_compute_instance" "$nombre" {
    enterprise_project_id = $enterprise_project_id
    name               = "$name"
    image_id           = "$image_id"
    flavor_id          = "$flavor_id"
    security_group_ids = ["$security_group_ids"]
    availability_zone  = "$availability_zone"

    system_disk_type = "$type"
    system_disk_size = $size

    dynamic "data_disks" {
        for_each = [{
            type = "$type"
            size = $size
        }]
        content {
            type = data_disks.value.type
            size = data_disks.value.size
        }
    }

    dynamic "network" {
        for_each = [{
            uuid = "$uuid"
        }]
        content {
            uuid = network.value.uuid
        }
    }
}
```

## 8. Instanciar user_data para windows

- **Prefijo:** `"tf-ecs-user-data`
- **Nota:** Este elemento se agrega dentro del bloque de `resource` de la ECS

```hcl
user_data = base64encode(file("$init_disk.ps1"))
```

## 9. Instanciar user_data linux block

- **Prefijo:** `tf-ecs-user-data-linux`
- **Nota:** Este elemento se agrega dentro del bloque de `resource` de la ECS

```hcl
user_data = base64encode(file("$cloud_init_script.yaml"))
```

## 10. Instanciar evs resource

- **Prefijo:** `tf-evs-resource`

```hcl
resource "huaweicloud_evs_volume" "$volumeName" {
    name              = "$volumeName"
    volume_type       = local.enum_evs.type.$Enum_type
    size              = $size
    availability_zone = "$availability_zone"
    server_id         = "$server_id_to_attach"
}
```

## 11. Instanciar EIP resource

- **Prefijo:** `tf-eip-resource`

```hcl
resource "huaweicloud_vpc_eip" "$eip_name" {
    publicip {
        type = local.enum_eip.type.$Enum_type
    }

    bandwidth {
        share_type  = local.enum_eip.share_type.$Enum_type
        name        = "$bandwidth_name"
        size        = $size
        charge_mode = local.enum_eip.charge_mode.$Enum_type
    }
}
```

## 12. Instanciar recurso de asociacion EIP con ECS

- **Prefijo:** `tf-eip-associate`

```hcl
resource "huaweicloud_compute_eip_associate" "$name" {
    public_ip   = "$address_ip"
    instance_id = "$ecs_id"
}
```

## 13. Instanciar VPC resource

- **Prefijo:** `tf-vpc-resource`

```hcl
resource "huaweicloud_vpc" "$vpc_name" {
    name = "$name"
    cidr = "$cidr_block"
}
```

## 14. Instanciar Subnet resource

- **Prefijo:** `tf-subnet-resource`

```hcl
resource "huaweicloud_vpc_subnet" "$subnet_name" {
    name              = "$subnet_name"
    cidr              = "$subnet_cidr"
    gateway_ip        = "$subnet_gateway_ip"
    vpc_id            = "$vpc_id"
    availability_zone = "$availability_zone"
}
```

## 15. Instanciar Peering resource

- **Prefijo:** `tf-vpc-peering`

```hcl
resource "huaweicloud_vpc_peering_connection" "$peering" {
    provider    = "huaweicloud.$mainAlias" # eliminar si el peering es con una VPC del mismo Tenant
    name        = "$peer_conn_name"
    vpc_id      = "$vpc_id"
    peer_vpc_id = "$accepter_vpc_id"
}
```

## 16. Instanciar Peering resource accepter

- **Prefijo:** `tf-vpc-peering-accepter`
- **Nota:** En el provider se debe incluir el provider con acceso al tenant remoto

```hcl
resource "huaweicloud_vpc_peering_connection_accepter" "$accepter" {
    provider                  = "huaweicloud.$peerAlias"
    accept                    = true
    vpc_peering_connection_id = huaweicloud_vpc_peering_connection.$peering.id
}
```

## 17. Instanciar Ruta resource

- **Prefijo:** `tf-route-resource`

```hcl
resource "huaweicloud_vpc_route" "$vpc_route" {
    vpc_id      = "$vpc_id"
    destination = "$cdir_destination"
    type        = local.enum_vpc.route_type.$Enum_type
    nexthop     = "$nexthop"
}
```

## 18. Instanciar Security Group resource

- **Prefijo:** `tf-secgroup-resource`

```hcl
resource "huaweicloud_networking_secgroup" "$secGroupName" {
    name                 = "$secGroupName"
    delete_default_rules = true
}
```

## 19. Instanciar Security Group Rule resource

- **Prefijo:** `tf-secgroup-rule-resource`

```hcl
resource "huaweicloud_networking_secgroup_rule" "$ruleName" {
    security_group_id         = "$security_group_id"
    action                    = local.enum_vpc.action_type.ALLOW
    priority                  = 1 # Eliminar si 1 por defecto
    direction                 = local.enum_vpc.secgroup_type.inbound
    ethertype                 = local.enum_vpc.ip_version.IPv4
    protocol                  = local.enum_vpc.protocol_type.TCP
    ports                     = $port1,$port2,$port3-$portN
    remote_ip_prefix          = "$cidr_block" # Eliminar si no es a un rango de IPs en específico
    remote_group_id           = "$remote_group_id" # Eliminar si no es a un Security Group
    remote_address_group_id   = "$remote_address_group_id" # Eliminar si no es a un grupo de IPs
}
```

## 20. Instanciar Address Group resource

- **Prefijo:** `tf-addr-group-resource`

```hcl
resource "huaweicloud_vpc_address_group" "$addrGroupName" {
    name       = "$addrGroupName"
    ip_version = 4

    ip_extra_set {
        ip      = "$ip_address"
        remarks = "$remarks_description"
    }

    ip_extra_set {
        ip      = "$cidr_block"
        remarks = "$remarks_description"
    }

    ip_extra_set {
        ip = "$ip_address1-$ip_addressN" # Rango de IPs
    }
}
```

## 21. Instanciar VPN Gateway resource

- **Prefijo:** `tf-vpn-gateway`

```hcl
data "huaweicloud_vpn_gateway_availability_zones" "$vpnName" {
    flavor          = local.enum_vpn.flavor.Professional1
    attachment_type = local.enum_vpn.attachment_type.VPC
}

resource "huaweicloud_vpn_gateway" "$vpnName" {
    enterprise_project_id = "$enterprise_project_id"
    name                  = "$vpnName"
    flavor                = local.enum_vpn.flavor.Professional1
    attachment_type       = local.enum_vpn.attachment_type.VPC
    vpc_id                = "$vpc_id"
    local_subnets         = ["$cidr_block_1", "$cidr_block_2"]
    connect_subnet        = "$subnet_id"
    asn                   = 64512 # 64,512 by default. Use 1 to 4,294,967,295
    ha_mode               = local.enum_vpn_ha_mode.Active-Standby
    availability_zones    = [
        data.huaweicloud_vpn_gateway_availability_zones.$vpnName.names[0],
        data.huaweicloud_vpn_gateway_availability_zones.$vpnName.names[1]
    ]

    eip1 {
        id             = "$eip1_id" # eliminar en caso de crear nueva
        bandwidth_name = "$bandwidth_name1"
        type           = local.enum_eip.type.dynamic-BGP
        bandwidth_size = $size
        charge_mode    = local.enum_eip.charge_mode.Traffic
    }

    eip2 {
        id             = "$eip2_id" # eliminar en caso de crear nueva
        bandwidth_name = "$bandwidth_name2"
        type           = local.enum_eip.type.dynamic-BGP
        bandwidth_size = $size
        charge_mode    = local.enum_eip.charge_mode.Traffic
    }
}
```

## 22. Instanciar VPN Customer Gateway resource

- **Prefijo:** `tf-vpn-cust-gw-resource`

```hcl
resource "huaweicloud_vpn_customer_gateway" "$custGwName" {
    name     = "$CustGwName"
    id_value = "$id_value" # IP o FQDN
    id_type  = local.enum_vpn.customer_id_type.IP
}
```

## 23. Instanciar VPN Connection static resource

- **Prefijo:** `tf-vpn-connection-static-resource`

```hcl
resource "huaweicloud_vpn_connection" "$vpnConnName" {
    name                = "$vpnConnName"
    gateway_id          = "$gateway_id"
    gateway_ip          = "$gateway_ip"
    customer_gateway_id = "$customer_gateway_id"
    peer_subnets        = ["$cidr_peer_subnet"]
    vpn_type            = local.enum_vpn.connection_type.Static-Routing
    psk                 = "$psk"

    ikepolicy {
        authentication_algorithm = local.enum_vpn.authentication_algorithm.sha2-256
        authentication_method    = local.enum_vpn.authentication_method.pre-share
        encryption_algorithm     = local.enum_vpn.encryption_algorithm.aes-128
        ike_version              = local.enum_vpn.ike_version.v2
        lifetime_seconds         = 86400
        dh_group                 = local.enum_vpn.dh_group.group15
    }

    ipsecpolicy {
        authentication_algorithm = local.enum_vpn.authentication_algorithm.sha2-256
        encapsulation_mode       = local.enum_vpn.encapsulation_mode.tunnel
        encryption_algorithm     = local.enum_vpn.encryption_algorithm.aes-128
        lifetime_seconds         = 3600
        pfs                      = local.enum_vpn.pfs.group14
        transform_protocol       = local.enum_vpn.transform_protocol.esp
    }
}
```

## 24. Instanciar VPN Connection bgp resource

- **Prefijo:** `tf-vpn-connection-bgp-resource`

```hcl
esource "huaweicloud_vpn_connection" "$vpnConnName" {
    name                = "$vpnConnName"
    gateway_id          = "$gateway_id"
    gateway_ip          = "$gateway_ip"
    customer_gateway_id = "$customer_gateway_id"
    peer_subnets        = ["$cidr_peer_subnet"]
    vpn_type            = local.enum_vpn.connection_type.BGP-Routing
    tunnel_local_address= "169.254.$pair.$addr1"
    tunnel_peer_address = "169.254.$pair.$addr2"
    psk                 = "$psk"

    ikepolicy {
        authentication_algorithm = local.enum_vpn.authentication_algorithm.sha2-256
        authentication_method    = local.enum_vpn.authentication_method.pre-share
        encryption_algorithm     = local.enum_vpn.encryption_algorithm.aes-128
        ike_version              = local.enum_vpn.ike_version.v2
        lifetime_seconds         = 86400
        dh_group                 = local.enum_vpn.dh_group.group15
    }

    ipsecpolicy {
        authentication_algorithm = local.enum_vpn.authentication_algorithm.sha2-256
        encapsulation_mode       = local.enum_vpn.encapsulation_mode.tunnel
        encryption_algorithm     = local.enum_vpn.encryption_algorithm.aes-128
        lifetime_seconds         = 3600
        pfs                      = local.enum_vpn.pfs.group14
        transform_protocol       = local.enum_vpn.transform_protocol.esp
    }
}
```

## 25. Instanciar VPN Connection policy resource

- **Prefijo:** `tf-vpn-connection-policy-resource`

```hcl
resource "huaweicloud_vpn_connection" "$vpnConnName" {
    name                = "$vpnConnName"
    gateway_id          = "$gateway_id"
    gateway_ip          = "$gateway_ip"
    customer_gateway_id = "$customer_gateway_id"
    peer_subnets        = ["$cidr_peer_subnet"]
    vpn_type            = local.enum_vpn.connection_type.Policy-Based
    psk                 = "$psk"

    policy_rules {
        source      = "$cidr_source"
        destination = ["$cidr_destination"]
    }

    ikepolicy {
        authentication_algorithm = local.enum_vpn.authentication_algorithm.sha2-256
        authentication_method    = local.enum_vpn.authentication_method.pre-share
        encryption_algorithm     = local.enum_vpn.encryption_algorithm.aes-128
        ike_version              = local.enum_vpn.ike_version.v2
        lifetime_seconds         = 86400
        dh_group                 = local.enum_vpn.dh_group.group15
    }

    ipsecpolicy {
        authentication_algorithm = local.enum_vpn.authentication_algorithm.sha2-256
        encapsulation_mode       = local.enum_vpn.encapsulation_mode.tunnel
        encryption_algorithm     = local.enum_vpn.encryption_algorithm.aes-128
        lifetime_seconds         = 3600
        pfs                      = local.enum_vpn.pfs.group14
        transform_protocol       = local.enum_vpn.transform_protocol.esp
    }
}
```
