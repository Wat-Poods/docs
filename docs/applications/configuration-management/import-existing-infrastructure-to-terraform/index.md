---
author:
  name: Linode
  email: docs@linode.com
description: 'This guide will describe how to import existing Linode infrastructure into Terraform using the official Linode provider plugin.'
keywords: ['terraform','configuration management','import']
license: '[CC BY-ND 4.0](https://creativecommons.org/licenses/by-nd/4.0)'
published: 2018-12-17
modified: 2018-12-17
modified_by:
  name: Linode
title: "Import Existing Infrastructure to Terraform"
contributor:
  name: Linode
external_resources:
- '[Terraform Import Usage](https://www.terraform.io/docs/import/usage.html)'
- '[Terraform Linode Instance Documentation](https://www.terraform.io/docs/providers/linode/r/instance.html)'
---

Terraform is an orchestration tool that uses declarative code to build, change, and version infrastructure that is made up of server instances and services. You can use [Linode's official Terraform provider](https://www.terraform.io/docs/providers/linode/index.html) to interact with Linode services. Existing Linode infrastructure can be imported and brought under Terraform management. This guide will describe how to import existing Linode infrastructure into Terraform using the official Linode provider plugin.

## Before You Begin

1.  Terraform and the Linode Terraform provider should be installed in your development environment. You should also have a basic understanding of [Terraform resources](https://www.terraform.io/docs/configuration/resources.html). To install and learn about Terraform, read our [Use Terraform to Provision Linode Environments](/docs/applications/configuration-management/how-to-build-your-infrastructure-using-terraform-and-linode/) guide.

2.  To use Terraform you must have a valid API access token. For more information on creating a Linode API access token, visit our [Getting Started with the Linode API](/docs/platform/api/getting-started-with-the-linode-api/#get-an-access-token) guide.

3.  This guide uses the Linode CLI to retrieve information about the Linode infrastructure you will import to Terraform. For more information on the setup, installation, and usage of the Linode CLI, check out the [Using the Linode CLI](https://www.linode.com/docs/platform/api/using-the-linode-cli/) guide.

## Terraform's Import Command

Throughout this guide the `terraform import` command will be used to import Linode resources. At the time of writing this guide, the import command **does not generate a Terraform resource configuration**. Instead, it imports your existing resources into Terraform's *state*.

State is Terraform's stored JSON mapping of your current Linode resources to their configurations. You can access and use the information provided by the state to manually create a corresponding resource configuration file and manage your existing Linode infrastructure with Terraform.

Additionally, there is no current way to import more than one resource at a time. **All resources must be individually imported**.

{{< caution >}}
When importing your infrastructure to Terraform, failure to accurately provide your Linode service's ID information can result in the unwanted alteration or destruction of the service. Please follow the instructions provided in this guide carefully. It might be beneficial to use multiple [Terraform Workspaces](https://www.terraform.io/docs/state/workspaces.html) to manage separate testing and production infrastructures.
{{< /caution >}}

## Import a Linode to Terraform

### Retrieve Your Linode's ID

1.  Using the Linode CLI, retrieve a list of all your Linode instances and find the ID of the Linode you would like to manage under Terraform:

        linode-cli linodes list --json --pretty

    {{< output >}}
[
&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;"id": 11426126,
&nbsp;&nbsp;&nbsp;&nbsp;"image": "linode/debian9",
&nbsp;&nbsp;&nbsp;&nbsp;"ipv4": [
&nbsp;&nbsp;&nbsp;&nbsp;"192.0.2.2"
&nbsp;&nbsp;&nbsp;&nbsp;],
&nbsp;&nbsp;&nbsp;&nbsp;"label": "terraform-import",
&nbsp;&nbsp;&nbsp;&nbsp;"region": "us-east",
&nbsp;&nbsp;&nbsp;&nbsp;"status": "running",
&nbsp;&nbsp;&nbsp;&nbsp;"type": "g6-standard-1"
&nbsp;&nbsp;}
]
{{< /output >}}

    This command will return a list of your existing Linodes in JSON format. From the list, find the Linode you would like to import and copy down its corresponding `id`. In this example, the Linode's ID is `11426126`. You will use your Linode's ID to import your Linode to Terraform.

### Create An Empty Resource Configuration

1. Ensure you are in your [Terraform project directory](/docs/applications/configuration-management/how-to-build-your-infrastructure-using-terraform-and-linode/#install-terraform). Create a Terraform configuration file to manage the Linode instance you will import in the next section. Your file can be named anything you like, but it must end in `.tf`. Add a Linode provider block with your API access token and an empty `linode_instance` resource configuration block in the file:

    {{< note >}}
The example resource block defines `example_label` as the label. This can be changed to any value you prefer. This label is used to reference your Linode resource configuration within Terraform, and does not have to be the same label originally assigned to the Linode when it was created outside of Terraform.
{{</ note >}}

    {{< file "linode_import.tf" >}}
provider "linode" {
    token = "your_API_access_token"
}

resource "linode_instance" "example_label" {}
{{< /file >}}

### Import Your Linode to Terraform

1. Run the `import` command, supplying the `linode_instance` resource's label, and the Linode's ID that was retrieved in the [Retrieve Your Linode's ID](/docs/applications/configuration-management/import-existing-infrastructure-to-terraform/#retrieve-your-linode-s-id) section :

        terraform import linode_instance.example_label linodeID

    You should see a similar output:

    {{< output >}}
linode_instance.example_label: Importing from ID "11426126"...
linode_instance.example_label: Import complete!
  Imported linode_instance (ID: 11426126)
linode_instance.example_label: Refreshing state... (ID: 11426126)

Import successful!

The resources that were imported are shown above. These resources are now in
your Terraform state and will henceforth be managed by Terraform.
{{< /output >}}

    This command will create a `terraform.tfstate` file with information about your Linode. You will use this information to fill out your resource configuration.

1.  To view the information created by `terraform import`, run the `show` command. This command will display a list of key-value pairs representing information about the imported Linode instance.

        terraform show

    You should see an output similar to the following:

    {{< output >}}
linode_instance.example_label:
  id = 11426126
  alerts.# = 1
  alerts.0.cpu = 90
  alerts.0.io = 10000
  alerts.0.network_in = 10
  alerts.0.network_out = 10
  alerts.0.transfer_quota = 80
  backups.# = 1
  boot_config_label = My Debian 9 Disk Profile
  config.# = 1
  config.0.comments =
  config.0.devices.# = 1
  config.0.devices.0.sda.# = 1
  config.0.devices.0.sda.0.disk_id = 24170011
  config.0.devices.0.sda.0.disk_label = Debian 9 Disk
  config.0.devices.0.sda.0.volume_id = 0
  config.0.devices.0.sdb.# = 1
  config.0.devices.0.sdb.0.disk_id = 24170012
  config.0.devices.0.sdb.0.disk_label = 512 MB Swap Image
  config.0.devices.0.sdb.0.volume_id = 0
  config.0.devices.0.sdc.# = 0
  config.0.devices.0.sdd.# = 0
  config.0.devices.0.sde.# = 0
  config.0.devices.0.sdf.# = 0
  config.0.devices.0.sdg.# = 0
  config.0.devices.0.sdh.# = 0
  config.0.helpers.# = 1
  config.0.helpers.0.devtmpfs_automount = true
  config.0.helpers.0.distro = true
  config.0.helpers.0.modules_dep = true
  config.0.helpers.0.network = true
  config.0.helpers.0.updatedb_disabled = true
  config.0.kernel = linode/grub2
  config.0.label = My Debian 9 Disk Profile
  config.0.memory_limit = 0
  config.0.root_device = /dev/root
  config.0.run_level = default
  config.0.virt_mode = paravirt
  disk.# = 2
  disk.0.authorized_keys.# = 0
  disk.0.filesystem = ext4
  disk.0.id = 24170011
  disk.0.image =
  disk.0.label = Debian 9 Disk
  disk.0.read_only = false
  disk.0.root_pass =
  disk.0.size = 50688
  disk.0.stackscript_data.% = 0
  disk.0.stackscript_id = 0
  disk.1.authorized_keys.# = 0
  disk.1.filesystem = swap
  disk.1.id = 24170012
  disk.1.image =
  disk.1.label = 512 MB Swap Image
  disk.1.read_only = false
  disk.1.root_pass =
  disk.1.size = 512
  disk.1.stackscript_data.% = 0
  disk.1.stackscript_id = 0
  group = Terraform
  ip_address = 192.0.2.2
  ipv4.# = 1
  ipv4.1835604989 = 192.0.2.2
  ipv6 = 2600:3c03::f03c:91ff:fef6:3ebe/64
  label = terraform-import
  private_ip = false
  region = us-east
  specs.# = 1
  specs.0.disk = 51200
  specs.0.memory = 2048
  specs.0.transfer = 2000
  specs.0.vcpus = 1
  status = running
  swap_size = 512
  type = g6-standard-1
  watchdog_enabled = true
{{< /output >}}

    You will use this information in the next section.

    {{< note >}}
There is a current bug in the Linode Terraform provider that causes the Linode's `root_device` configuration to display an import value of `/dev/root`, instead of `/dev/sda`. This is visible in the example output above: `config.0.root_device = /dev/root`. However, the correct disk, `/dev/sda`, is in fact targeted. For this reason, when running the `terraform plan` or the `terraform apply` commands, the output will display `config.0.root_device: "/dev/root" => "/dev/sda"`.

You can follow the corresponding [GitHub issue](https://github.com/terraform-providers/terraform-provider-linode/issues/10) for more details.
{{< /note >}}

### Fill In Your Linode's Configuration Data

As mentioned in the [Terraform's Import Command](/docs/applications/configuration-management/import-existing-infrastructure-to-terraform/#terraform-s-import-command) section, you must manually create your resource configurations when importing existing infrastructure.

  1. Fill in the configuration values for the `linode_instance` resource block. In the example below, the necessary values were collected from the output of the `terraform show` command applied in Step 2 of the [Import Your Linode to Terraform](/docs/applications/configuration-management/import-existing-infrastructure-to-terraform/#import-your-linode-to-terraform) section. The file's comments indicate the corresponding keys used to determine the values for the `linode_instance` configuration block.

      {{< file "linode_instance_import.tf" >}}
provider "linode" {
    token = "a12b3c4e..."
}

resource "linode_instance" "example_label" {
    label = "terraform-import" #label
    region = "us-east"         #region
    type = "g6-standard-1"     #type
    config {
        label = "My Debian 9 Disk Profile"     #config.0.label
        kernel = "linode/grub2"                #config.0.kernel
        root_device = "/dev/sda"               #config.0.root_device
        devices {
            sda = {
                disk_label = "Debian 9 Disk"    #config.0.devices.0.sda.0.disk_label
            }
            sdb = {
                disk_label = "512 MB Swap Image" #config.0.devices.0.sdb.0.disk_label
            }
        }
    }
    disk {
        label = "Debian 9 Disk"      #disk.0.label
        size = "50688"               #disk.0.size
    }
    disk {
        label = "512 MB Swap Image"  #disk.1.label
        size = "512"                 #disk.1.size
    }
}
    {{< /file >}}

    {{< note >}}
If your Linode uses more than two disks (for instance, if you have attached a [Block Storage Volume](/docs/platform/block-storage/how-to-use-block-storage-with-your-linode-new-manager/)), you will need to add those disks to your Linode resource configuration block. In order to add a disk, you must add the disk to the `devices` stanza and create an additional `disk` stanza.
{{< /note >}}

    {{< note >}}
If you have more than one [configuration profile](/docs/platform/disk-images/disk-images-and-configuration-profiles/), you must choose which profile to boot from with the `boot_config_label` argument. For example:

    resource "linode_instance" "example_label" {
        boot_config_label = "My Debian 9 Disk Profile"
    ...
{{< /note >}}

1.  To check for errors in your configuration, run the `plan` command:

        terraform plan

    `terraform plan` shows you the changes that would take place if you were to apply the configurations with a `terraform apply`. Running `terraform plan` is a good way to determine if the configuration you provided is exact enough for Terraform to take over the management of your Linode.

    {{< note >}}
  Running `terraform plan` will display any changes that will be applied to your existing infrastructure based on your configuration file(s). However, you will **not be notified** about the **addition and removal of disks** with `terraform plan`. For this reason, it is vital that the values you include in your `linode_instance` resource configuration block match the values generated from running the `terraform show` command.
    {{</ note >}}

1. Once you have verified the configurations you provided in the `linode_instance` resource block, you are ready to begin managing your Linode instance with Terraform. Any changes or updates can be made by updating your `linode_instance_import.tf` file, then verifying the changes with the `terrform plan` command, and then finally applying the changes with the `terraform apply` command.

    For more available configuration options, visit the [Linode Instance](https://www.terraform.io/docs/providers/linode/r/instance.html) Terraform documentation.

## Import a Domain to Terraform

### Retrieve Your Domain's ID

1. Using the Linode CLI, retrieve a list of all your domains to find the ID of the domain you would like to manage under Terraform:

        linode-cli domains list --json --pretty

    You should see output like the following:

    {{< output >}}
[
&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;"domain": "import-example.com",
&nbsp;&nbsp;&nbsp;&nbsp;"id": 1157521,
&nbsp;&nbsp;&nbsp;&nbsp;"soa_email": "webmaster@import-example.com",
&nbsp;&nbsp;&nbsp;&nbsp;"status": "active",
&nbsp;&nbsp;&nbsp;&nbsp;"type": "master"
&nbsp;&nbsp;}
]
{{< /output >}}

    Find the domain you would like to import and copy down the ID. You will need this ID to import your domain to Terraform.

### Create an Empty Resource Configuration

1. Ensure you are in your [Terraform project directory](/docs/applications/configuration-management/how-to-build-your-infrastructure-using-terraform-and-linode/#install-terraform). Create a Terraform configuration file to manage the domain you will import in the next section. Your file can be named anything you like, but must end in `.tf`. Add a Linode provider block with your API access token and an empty `linode_domain` resource configuration block to the file:

    {{< file "domain_import.tf" >}}
provider "linode" {
    token = "Your API Token"
}

resource "linode_domain" "example_label" {}
{{< /file >}}

### Import Your Domain to Terraform

1. Run the `import` command, supplying the `linode_domain` resource's label, and the domain ID that was retrieved in the [Retrieve Your Domain's ID](/docs/applications/configuration-management/import-existing-infrastructure-to-terraform/#retrieve-your-domain-s-id) section:

        terraform import linode_domain.example_label domainID

    You should see output similar to the following:

    {{< output >}}
linode_domain.example_label: Importing from ID "1157521"...
linode_domain.example_label: Import complete!
  Imported linode_domain (ID: 1157521)
linode_domain.example_label: Refreshing state... (ID: 1157521)

Import successful!

The resources that were imported are shown above. These resources are now in
your Terraform state and will henceforth be managed by Terraform.
{{< /output >}}

      This command will create a `terraform.tfstate` file with information about your domain. You will use this information to fill out your resource configuration.

1. To view the information created by `terraform import`, run the show command. This command will display a list of key-value pairs representing information about the imported domain:

        terraform show

    You should see output like the following:

    {{< output >}}
linode_domain.example_label:
  id = 1157521
  description =
  domain = import-example.com
  expire_sec = 0
  group =
  master_ips.# = 0
  refresh_sec = 0
  retry_sec = 0
  soa_email = webmaster@import-example.com
  status = active
  ttl_sec = 0
  type = master
{{< /output >}}

### Fill In Your Domain's Configuration Data

As mentioned in the [Terraform’s Import Command](/docs/applications/configuration-management/import-existing-infrastructure-to-terraform/#terraform-s-import-command) section, you must manually create your resource configurations when importing existing infrastructure.

1. Fill in the configuration values for the `linode_domain` resource block. The necessary values for the example resource configuration file were collected from the output of the `terraform show` command applied in Step 2 of the [Import Your Domain to Terraform](/docs/applications/configuration-management/import-existing-infrastructure-to-terraform/#import-your-domain-to-terraform) section.

    {{< file "linode_domain_example.tf" >}}
provider "linode" {
    token = "1a2b3c..."
}

resource "linode_domain" "example_label" {
    domain = "import-example.com"
    soa_email = "webmaster@import-example.com"
    type = "master"
}
    {{< /file >}}

    {{< note >}}
  If your Domain `type` is `slave` then you'll need to include a `master_ips` argument with values set to the IP addresses that represent the Master DNS for your domain.
    {{< /note >}}

1. Check for errors in your configuration by running the `plan` command:

        terraform plan

    `terraform plan` shows you the changes that would take place if you were to apply the configurations with the `terraform apply` command. Running `terraform plan` should result in Terraform displaying that no changes are to be made.

1. Once you have verified the configurations you provided in the `linode_domain` block, you are ready to begin managing your domain with Terraform. Any changes or updates can be made by updating your `linode_domain_example.tf` file, then verifying the changes with the `terrform plan` command, and then finally applying the changes with the `terraform apply` command.

    For more available configuration options, visit the [Linode Domain](https://www.terraform.io/docs/providers/linode/r/domain.html) Terraform documentation.

## Import a Block Storage Volume to Terraform

### Retrieve Your Block Storage Volume's ID

1. Using the Linode CLI, retrieve a list of all your volumes to find the ID of the Block Storage Volume you would like to manage under Terraform:

        linode-cli volumes list --json --pretty

    You should see output similar to the following:

    {{< output >}}
[
&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;"id": 17045,
&nbsp;&nbsp;&nbsp;&nbsp;"label": "import-example",
&nbsp;&nbsp;&nbsp;&nbsp;"linode_id": 11426126,
&nbsp;&nbsp;&nbsp;&nbsp;"region": "us-east",
&nbsp;&nbsp;&nbsp;&nbsp;"size": 20,
&nbsp;&nbsp;&nbsp;&nbsp;"status": "active"
&nbsp;&nbsp;}
]
{{< /output >}}

    Find the Block Storage Volume you would like to import and copy down the ID. You will use this ID to import your volume to Terraform.

### Create an Empty Resource Configuration

1. Ensure you are in your Terraform project directory. Create a Terraform configuration file to manage the Block Storage Volume you will import in the next section. Your file can be named anything you like, but must end in `.tf`. Add a Linode provider block with your API access token and an empty `linode_volume` resource configuration block to the file:

    {{< file "linode_volume_example.tf" >}}
provider "linode" {
    token = "Your API Token"
}

resource "linode_volume" "example_label" {}
{{< /file >}}

### Import Your Volume to Terraform

1. Run the `import` command, supplying the `linode_volume` resource's label, and the volume ID that was retrieved in the [Retrieve Your Block Storage Volume's ID](/docs/applications/configuration-management/import-existing-infrastructure-to-terraform/#retrieve-your-block-storage-volume-s-id) section:

        terraform import linode_volume.example_label volumeID

    You should see output similar to the following:

    {{< output >}}
linode_volume.example_label: Importing from ID "17045"...
linode_volume.example_label: Import complete!
  Imported linode_volume (ID: 17045)
linode_volume.example_label: Refreshing state... (ID: 17045)

Import successful!

The resources that were imported are shown above. These resources are now in
your Terraform state and will henceforth be managed by Terraform.
{{< /output >}}

    This command will create a `terraform.tfstate` file with information about your Volume. You will use this information to fill out your resource configuration.

1.  To view the information created by `terraform import`, run the `show` command. This command will display a list of key-value pairs representing information about the imported Volume:

        terraform show

    You should see output like the following:

    {{< output >}}
linode_volume.example_label:
  id = 17045
  filesystem_path = /dev/disk/by-id/scsi-0Linode_Volume_import-example
  label = import-example
  linode_id = 11426126
  region = us-east
  size = "20"
  status = active
{{< /output >}}

### Fill In Your Volume's Configuration Data

As mentioned in the [Terraform’s Import Command](/docs/applications/configuration-management/import-existing-infrastructure-to-terraform/#terraform-s-import-command) section, you must manually create your resource configurations when importing existing infrastructure.

1. Fill in the configuration values for the `linode_volume` resource block. The necessary values for the example resource configuration file were collected from the output of the `terraform show` command applied in Step 2 of the [Import Your Volume to Terraform](/docs/applications/configuration-management/import-existing-infrastructure-to-terraform/#import-your-volume-to-terraform) section:

    {{< file "linode_volume_example.tf" >}}
provider "linode" {
    token = "1a2b3c..."
}

resource "linode_volume" "example_label" {
    label = "import-example"
    region = "us-east"
    size = "20"
}
    {{< /file >}}

    {{< note >}}
Though it is not required, it's a good idea to include a configuration for the size of the volume so that it can be managed more easily should you ever choose to expand the Volume. It is not possible to reduce the size of a volume.
    {{< /note >}}

1. Check for errors in your configuration by running the `plan` command:

        terraform plan

    `terraform plan` shows you the changes that would take place if you were to apply the configurations with the `terraform apply` command. Running `terraform plan` should result in Terraform displaying that no changes are to be made.

1. Once you have verified the configurations you provided in the `linode_volume` block, you are ready to begin managing your Block Storage Volume with Terraform. Any changes or updates can be made by updating your `linode_volume_example.tf` file, then verifying the changes with the `terrform plan` command, and then finally applying the changes with the `terraform apply` command.

    For more optional configuration options, visit the [Linode Volume](https://www.terraform.io/docs/providers/linode/r/volume.html) Terraform documentation.

## Import a NodeBalancer to Terraform

Configuring [Linode NodeBalancers](/docs/platform/nodebalancer/getting-started-with-nodebalancers/) with Terraform requires three separate resource configuration blocks: one to create the NodeBalancer, a second for the NodeBalancer Configuration, and a third for the NodeBalancer Nodes.

### Retrieve Your NodeBalancer, NodeBalancer Config, NodeBalancer Node IDs

1. Using the Linode CLI, retrieve a list of all your NodeBalancers to find the ID of the NodeBalancer you would like to manage under Terraform:

        linode-cli nodebalancers list --json --pretty

    You should see output similar to the following:

    {{< output >}}
[
&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;"client_conn_throttle": 0,
&nbsp;&nbsp;&nbsp;&nbsp;"hostname": "nb-192-0-2-3.newark.nodebalancer.linode.com",
&nbsp;&nbsp;&nbsp;&nbsp;"id": 40721,
&nbsp;&nbsp;&nbsp;&nbsp;"ipv4": "192.0.2.3",
&nbsp;&nbsp;&nbsp;&nbsp;"ipv6": "2600:3c03:1::68ed:945f",
&nbsp;&nbsp;&nbsp;&nbsp;"label": "terraform-example",
&nbsp;&nbsp;&nbsp;&nbsp;"region": "us-east"
&nbsp;&nbsp;}
]
{{< /output >}}

    Find the NodeBalancer you would like to import and copy down the ID. You will use this ID to import your NodeBalancer to Terraform.

1. Retrieve your NodeBalancer configuration by supplying the ID of the NodeBalancer you retrieved in the previous step:

        linode-cli nodebalancers configs-list 40721 --json --pretty

    You should see output similar to the following:

    {{< output >}}
[
&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;"algorithm": "roundrobin",
&nbsp;&nbsp;&nbsp;&nbsp;"check_passive": true,
&nbsp;&nbsp;&nbsp;&nbsp;"cipher_suite": "recommended",
&nbsp;&nbsp;&nbsp;&nbsp;"id": 35876,
&nbsp;&nbsp;&nbsp;&nbsp;"port": 80,
&nbsp;&nbsp;&nbsp;&nbsp;"protocol": "http",
&nbsp;&nbsp;&nbsp;&nbsp;"ssl_commonname": "",
&nbsp;&nbsp;&nbsp;&nbsp;"ssl_fingerprint": "",
&nbsp;&nbsp;&nbsp;&nbsp;"stickiness": "table"
&nbsp;&nbsp;}
]
{{< /output >}}

    Copy down the ID of your NodeBalancer configuration, you will use it to import your NodeBalancer configuration to Terraform.

1. Retrieve a list of Nodes corresponding to your NodeBalancer to find the label and address of your NodeBalancer Nodes. Supply the ID of your NodeBalancer as the first argument and the ID of your NodeBalancer configuration as the second:

        linode-cli nodebalancers nodes-list 40721 35876 --json --pretty

    You should see output like the following:

    {{< output >}}
[
&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;"address": "192.168.214.37:80",
&nbsp;&nbsp;&nbsp;&nbsp;"id": 327539,
&nbsp;&nbsp;&nbsp;&nbsp;"label": "terraform-import",
&nbsp;&nbsp;&nbsp;&nbsp;"mode": "accept",
&nbsp;&nbsp;&nbsp;&nbsp;"status": "UP",
&nbsp;&nbsp;&nbsp;&nbsp;"weight": 100
&nbsp;&nbsp;}
]
{{< /output >}}

    If you are importing a NodeBalancer, chances are your output lists more than one Node. Copy down the IDs of each Node. You will use them to import your Nodes to Terraform.

### Create Empty Resource Configurations

1. Ensure you are in your Terraform project directory. Create a Terraform configuration file to manage the NodeBalancer you will import in the next section. Your file can be named anything you like, but must end in `.tf`.

    Add a Linode provider block with your API access token and empty `linode_nodebalancer`, `linode_nodebalancer_config`, and `linode_nodebalancer_node` resource configuration blocks to the file. Be sure to give the resources appropriate labels. These labels will be used to reference the resources locally within Terraform:

    {{< file "linode_nodebalancer_example.tf" >}}
provider "linode" {
    token = "Your API Token"
}

resource "linode_nodebalancer" "example_nodebalancer_label" {}

resource "linode_nodebalancer_config" "example_nodebalancer_config_label" {}

resource "linode_nodebalancer_node" "example_nodebalancer_node_label" {}
{{< /file >}}

    If you have more than one NodeBalancer Configuration, you will need to supply multiple `linode_nodebalancer_config` resource blocks with different labels. The same is true for each NodeBalancer Node requiring an additional `linode_nodebalancer_node` block.

### Import Your NodeBalancer, NodeBalancer Configuration, and NodeBalancer Nodes to Terraform

1.  Run the `import` command for your NodeBalancer, supplying your local label and the ID of your NodeBalancer as the last parameter.

        terraform import linode_nodebalancer.example_nodebalancer_label nodebalancerID

    You should see output similar to the following:

    {{< output >}}
linode_nodebalancer.example_nodebalancer_label: Importing from ID "40721"...
linode_nodebalancer.example_nodebalancer_label: Import complete!
  Imported linode_nodebalancer (ID: 40721)
linode_nodebalancer.example_nodebalancer_label: Refreshing state... (ID: 40721)

Import successful!

The resources that were imported are shown above. These resources are now in
your Terraform state and will henceforth be managed by Terraform.
{{< /output >}}

1.  Run the `import` command for your NodeBalancer configuration, supplying your local label, and the ID of your NodeBalancer and the ID of your NodeBalancer configuration separated by commas as the last argument.

        terraform import linode_nodebalancer_config.example_nodebalancer_config_label nodebalancerID,nodebalancerconfigID

    You should see output similar to the following:

    {{< output >}}
linode_nodebalancer_config.example_nodebalancer_config_label: Importing from ID "40721,35876"...
linode_nodebalancer_config.example_nodebalancer_config_label: Import complete!
  Imported linode_nodebalancer_config (ID: 35876)
linode_nodebalancer_config.example_nodebalancer_config_label: Refreshing state... (ID: 35876)

Import successful!

The resources that were imported are shown above. These resources are now in
your Terraform state and will henceforth be managed by Terraform.
{{< /output >}}

1.  Run the `import` command for you NodeBalancer Nodes, supplying your local label, and the ID of your NodeBalancer, the ID of your NodeBalancer Configuration, and your NodeBalancer Node, separated by commas, as the last argument.

        terraform import linode_nodebalancer_node.example_nodebalancer_node_label nodebalancerID,nodebalancerconfigID,nodebalancernodeID


    You should see output like the following:

    {{< output >}}
linode_nodebalancer_node.example_nodebalancer_node_label: Importing from ID "40721,35876,327539"...
linode_nodebalancer_node.example_nodebalancer_node_label: Import complete!
  Imported linode_nodebalancer_node (ID: 327539)
linode_nodebalancer_node.example_nodebalancer_node_label: Refreshing state... (ID: 327539)

Import successful!

The resources that were imported are shown above. These resources are now in
your Terraform state and will henceforth be managed by Terraform.
{{< /output >}}


1.  Running `terraform import` creates a `terraform.tfstate` file with information about your NodeBalancer. You will use this information to fill out your resource configuration. To view the information created by `terraform import`, run the `show` command:

        terraform show

    You should see output like the following:

    {{< output >}}
linode_nodebalancer.example_nodebalancer_label:
  id = 40721
  client_conn_throttle = 0
  created = 2018-11-16T20:21:03Z
  hostname = nb-192-0-2-3.newark.nodebalancer.linode.com
  ipv4 = 192.0.2.3
  ipv6 = 2600:3c03:1::68ed:945f
  label = terraform-example
  region = us-east
  transfer.% = 3
  transfer.in = 0.013627052307128906
  transfer.out = 0.0015048980712890625
  transfer.total = 0.015131950378417969
  updated = 2018-11-16T20:21:03Z

linode_nodebalancer_config.example_nodebalancer_config_label:
  id = 35876
  algorithm = roundrobin
  check = none
  check_attempts = 2
  check_body =
  check_interval = 5
  check_passive = true
  check_path =
  check_timeout = 3
  cipher_suite = recommended
  node_status.% = 2
  node_status.down = 0
  node_status.up = 1
  nodebalancer_id = 40721
  port = 80
  protocol = http
  ssl_commonname =
  ssl_fingerprint =
  ssl_key =
  stickiness = table

linode_nodebalancer_node.example_nodebalancer_node_label:
  id = 327539
  address = 192.168.214.37:80
  config_id = 35876
  label = terraform-import
  mode = accept
  nodebalancer_id = 40721
  status = UP
  weight = 100
{{< /output >}}

### Fill In Your NodeBalancer's Configuration Data

As mentioned in the [Terraform’s Import Command](/docs/applications/configuration-management/import-existing-infrastructure-to-terraform/#terraform-s-import-command) section, you must manually create your resource configurations when importing existing infrastructure.

1. Fill in the configuration values for all three NodeBalancer resource configuration blocks. The necessary values for the example resource configuration file were collected from the output of the `terraform show` command applied in Step 4 of the [Import Your NodeBalancer, NodeBalancer Configuration, and NodeBalancer Nodes to Terraform](/docs/applications/configuration-management/import-existing-infrastructure-to-terraform/#import-your-nodebalancer-nodebalancer-configuration-and-nodebalancer-nodes-to-terraform) section:

{{< file "linode_nodebalancer_example.tf" >}}
provider "linode" {
    token = "1a2b3c..."
}

resource "linode_nodebalancer" "nodebalancer_import" {
    label = "terraform-example"
    region = "us-east"
}

resource "linode_nodebalancer_config" "nodebalancer_config_import" {
    nodebalancer_id = "40721"
}

resource "linode_nodebalancer_node" "nodebalancer_node_import" {
    label = "terraform-import"
    address = "192.168.214.37:80"
    nodebalancer_id = "40721"
    config_id = "35876"
}
{{< /file >}}

1. Check for errors in your configuration by running the `plan` command:

        terraform plan

    `terraform plan` shows you the changes that would take place if you were to apply the configurations with the `terraform apply` command. Running `terraform plan` should result in Terraform displaying that no changes are to be made.

1. Once you have verified the configurations you provided in all three NodeBalancer configuration blocks, you are ready to begin managing your NodeBalancers with Terraform. Any changes or updates can be made by updating your `linode_nodebalancer_example.tf` file, then verifying the changes with the `terrform plan` command, and finally, applying the changes with the `terraform apply` command.

    For more available configuration options, visit the [Linode NodeBalancer](https://www.terraform.io/docs/providers/linode/r/nodebalancer.html), [Linode NodeBalancer Config](https://www.terraform.io/docs/providers/linode/r/nodebalancer_config.html), and [Linode NodeBalancer Node](https://www.terraform.io/docs/providers/linode/r/nodebalancer_node.html) Terraform documentation.

## Next Steps

You can follow a process similar to what has been outlined in this guide to begin importing other pieces of your Linode infrastructure such as images, SSH keys, access tokens, and StackScripts. Check out the links in the [More Information](/docs/applications/configuration-management/import-existing-infrastructure-to-terraform/#more-information) section below for helpful information.
