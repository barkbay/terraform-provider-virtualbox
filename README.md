# **[This project is not actively maintained, someone please take over it.]**

# VirtualBox provider for Terraform

Inspired by [terraform-provider-vix](https://github.com/hooklift/terraform-provider-vix)

Fork from [terraform-provider-virtualbox](https://github.com/ccll/terraform-provider-virtualbox) by ccll

# How to install

1. go get github.com/pyToshka/terraform-provider-virtualbox

# How to build from source

1. git clone https://github.com/pyToshka/terraform-provider-virtualbox
2. cd terraform-provider-virtualbox
3. go get
4. mv terraform-provider-virtualbox example/
5. cd example/
6. terraform plan
7. terraform apply

# Resources

## "virtualbox_vm"

### Schema

- `name`, string, required: The name of the virtual machine.
- `image`, string, required: The place  of the image file(archive or vagrant box).
- `url`, string, optional, default not set: The url for downloaded vagrant box from external resource (ex. [Ubuntu Vagrant box](https://atlas.hashicorp.com/ubuntu/boxes/trusty64/versions/14.04/providers/virtualbox.box])) . If not set using `image` variable.
- `cpus`, int, optional, default=2: The number of CPUs.
- `memory`, string, optional, default="512mib": The size of memory, allow human friendly units like 'MB', 'MiB'.
- `user_data`, string, optional, default="": User defined data.
- `status`, string, optional, default="running": The status of the VM, allowed values: 'poweroff', 'running'. This value will be updated at runtime to reflect the real status of the VM, and you can also specify it explicitly in config to manually control the status of the VM. This value defaults to 'running', so `terraform apply` will always try to keep the VM running if not specified otherwise.
- `network_adapter`, list: The network adapters in the VM, you can have up to 4 adapters.
  - `.#.type`, string, requried: The type of the network, allowed values: 'nat', 'bridged', 'hostonly', 'internal', 'generic'.
  - `.#.device`, string, optional, default="IntelPro1000MTServer": The model of the virtual hardware device, allowed values: 'PCIII', 'FASTIII', 'IntelPro1000MTDesktop', 'IntelPro1000TServer', 'IntelPro1000MTServer'.
  - `.#.host_interface`, string, optional: Some network type (hostonly, bridged, etc) must bind to a host interface to work properly, use this field to specify the name of the host interface you like to bind to (like 'en0', 'eth1', 'wlan', etc). This should get an improvement, see [TODO](#todo) section below.
  - `.#.status`, string, computed: The status of the network adapter, possible values: 'up', 'down'.
  - `.#.mac_address`, string, computed: The MAC address of the adapter, this is generated by VirtualBox.
  - `.#.ipv4_address`, string, computed: The IPv4 address assigned to the adapter.
  - `.#.ipv4_address_available`, string, computed: Wheather or not an IPv4 address is actaully assigned to the adapter, possible values: "yes", "no".

### Network adapter types
- [x] NAT
- [x] bridged
- [ ] Hostonly  (not tested, probably can work)
- [ ] Internal  (not tested)
- [ ] Generic  (not tested)

# Example

```
resource "virtualbox_vm" "node" {
    count = 2
    name = "${format("node-%02d", count.index+1)}"

    image = "~/ubuntu-15.04.tar.xz"
    cpus = 2
    memory = "512mib"

    network_adapter {
        type = "nat"
    }

    network_adapter {
        type = "bridged"
        host_interface = "en0"
    }
}

output "IPAddr" {
    # Get the IPv4 address of the bridged adapter (the 2nd one) on 'node-02'
    value = "${element(virtualbox_vm.node.*.network_adapter.1.ipv4_address, 1)}"
}

```

# Limitations

- Experimental provider!

# Example images

- [ubuntu-15.04](https://github.com/ccll/terraform-provider-virtualbox-images/releases/tag/ubuntu-15.04)

- [Ubuntu Vagrant box](https://atlas.hashicorp.com/ubuntu/boxes/trusty64/versions/14.04/providers/virtualbox.box])


# TODO

- [ ] Optimize resourceVMUpdate(), eliminate unneccessary restarts of VM.
- [x] Auto download image from remote url.
- [ ] Validate downloaded image against checksum.
- [ ] Download the same image only once (based on checksum).
- [ ] Re-download corrupted image (based on checksum).
- [ ] Eliminate the mandatory usage of 'host_interface' in config file. For now the 'host_interface' must be specified in 'hostonly' and 'bridged' network, and must reference a valid host interface, otherwise the VM will refuse to boot. This is bad for team collaboration as the host interface names are often different on different machines. A more robust approach is to enumerate all host interfaces and automatically assign one (maybe based on default routing information?) if not specified explicitly.
