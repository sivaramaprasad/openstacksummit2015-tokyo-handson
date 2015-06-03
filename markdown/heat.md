# Heat
(not an acronym)


# Heat
enables you to deploy
## complete
virtual environments


## Heat
supports two distinct
# formats


# CFN
Amazon CloudFormation compatible template
# HOT
Heat Orchestration Template


HOT is 100%
# YAML


What can we
# do
with Heat?


## `OS::Nova::Server`
Configures Nova guests


```yaml
resources:
  my_box:
    type: OS::Nova::Server
    properties:
      name: my_box
      image: trusty-server-cloudimg-amd64
      flavor: m1.tiny
      key_name: my_key
```


Now we could just
# create
this stack


### `heat stack-create -f stack.yml`


But as it is,

it's not very
# flexible


Let's add some
## parameters


```yaml
parameters:
  flavor:
    type: string
    description: Flavor to use for servers
    default: m1.medium
  image:
    type: string
    description: Image name or ID
    default: trusty-server-cloudimg-amd64
  key_name:
    type: string
    description: Keypair to inject into newly created servers
```


And some
## intrinsic functions


```yaml
resources:
  my_box:
    type: OS::Nova::Server
    properties:
      name: my_box
      image: { get_param: image }
      flavor: { get_param: flavor }
```


And now we can
# set
these parameters


```sh
heat stack-create -f stack.yml` \
  -P key_name=mykey
  -P image=cirros-0.3.3-x86_64
```


How about we add some
## network connectivity
Wouldn't that be nice?


### `OS::Neutron::Net`
### `OS::Neutron::Subnet`
Defines Neutron networks


```yaml
  my_net:
    type: OS::Neutron::Net
    properties:
      name: management-net

  my_sub_net:
    type: OS::Neutron::Subnet
    properties:
      name: management-sub-net
      network_id: { get_resource: management_net }
      cidr: 192.168.122.0/24
      gateway_ip: 192.168.122.1
      enable_dhcp: true
      allocation_pools: 
        - start: "192.168.122.2"
          end: "192.168.122.50"
```


## `get_resource`
Cross-reference between resources

Automatic dependency


### `OS::Neutron::Router`
### `OS::Neutron::RouterGateway`
### `OS::Neutron::RouterInterface`
Configures Neutron routers


```yaml
parameters:
  public_net_id:
    type: string
    description: Public network ID

resources:
  router:
    type: OS::Neutron::Router
  router_gateway:
    type: OS::Neutron::RouterGateway
    properties:
      router_id: { get_resource: router }
      network_id: { get_param: public_net_id }
  router_interface:
    type: OS::Neutron::RouterInterface
    properties:
      router_id: { get_resource: router }
      subnet_id: { get_resource: my_sub_net }
```


### `OS::Neutron::Port`
Configures Neutron ports


```yaml
  mybox_management_port:
    type: OS::Neutron::Port
    properties:
      network_id: { get_resource: my_net }
```

```yaml
  my_box:
    type: OS::Nova::Server
    properties:
      name: deploy
      image: { get_param: image }
      flavor: { get_param: flavor }
      key_name: { get_param: key_name }
      networks:
        - port: { get_resource: mybox_management_port }
```


### `OS::Neutron::FloatingIP`
Allocates floating IP addresses



```yaml
  my_floating_ip:
    type: OS::Neutron::FloatingIP
    properties:
      floating_network_id: { get_param: public_net_id }
      port_id: { get_resource: mybox_management_port }
```


Integrating
# Heat
with
## `cloud-init`


```yaml
  my_box:
    type: OS::Nova::Server
    properties:
      name: deploy
      image: { get_param: image }
      flavor: { get_param: flavor }
      key_name: { get_param: key_name }
      networks:
        - port: { get_resource: mybox_management_port }
      user_data: { get_file: cloud-config.yml }
      user_data_format: RAW
```


### `OS::Heat::CloudConfig`
Manages `cloud-config` directly from Heat


```yaml
resources:
  my_config:
    type: "OS::Heat::CloudConfig"
    properties:
      cloud_config:
        package_update: true
        package_upgrade: true
```

```yaml
  my_box:
    type: OS::Nova::Server
    properties:
      name: deploy
      image: { get_param: image }
      flavor: { get_param: flavor }
      key_name: { get_param: key_name }
      networks:
        - port: { get_resource: mybox_management_port }
      user_data: { get_resource: my_config }
      user_data_format: RAW
```

