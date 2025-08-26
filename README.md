# Terraform-OCI-Free

This is a Terraform configuration that creates the Always Free instance on Oracle Cloud Infrastructure. By default, instance will have 4 Arm-based Ampere A1 cores, 24 GB of memory and 200 GB volume size.

In order to use Oracle Cloud Free Tier, you'll need to register free tier account. Once you have that set up, you can proceed with configuring auth.

## Configure auth

Generate API key pair that will be used by provider.

```bash
mkdir ~/.oci
openssl genrsa -out ~/.oci/oci.pem 4096
openssl rsa -pubout -in ~/.oci/oci.pem -out ~/.oci/oci_public.pem
```

Go to `Profile >> My profile >> API keys` and `Add API key`. Paste public key and copy configuration preview file. Save configuration file as `terraform.tfvars` in repository root.

### Optionally

Generate ed25519 key pair that will be used to SSH into instance.

For more details, check [docs](https://docs.oracle.com/en-us/iaas/Content/Identity/Tasks/managingcredentials.htm).

```bash
ssh-keygen -t ed25519 -C oci -f ~/.ssh/oci_free
```

## Configuration

### 1. Create terraform.tfvars

Create a `terraform.tfvars` file with your OCI configuration:

```hcl
user                     = "ocid1.user.oc1..your_user_ocid"
fingerprint              = "your:api:key:fingerprint"
tenancy                  = "ocid1.tenancy.oc1..your_tenancy_ocid"
region                   = "your_region"
namespace                = "your_object_storage_namespace"
key_file                 = "~/.oci/oci.pem"
instance_public_key_path = "~/.ssh/oci_free.pub"  # Optional: leave empty to auto-generate
```

### 2. Configure Backend

Create a `state.config` file for backend configuration:

```hcl
bucket           = "tfstate"
key              = "free/terraform.tfstate"
region           = "your_region"
namespace        = "your_object_storage_namespace"
user_ocid        = "ocid1.user.oc1..your_user_ocid"
fingerprint      = "your:api:key:fingerprint"
tenancy_ocid     = "ocid1.tenancy.oc1..your_tenancy_ocid"
private_key_path = "~/.oci/oci.pem"
```

## Create instance
```bash
terraform init -backend-config=state.config
terraform plan -var-file=terraform.tfvars
terraform apply -var-file=terraform.tfvars
```

## Connect to instance

```bash
# Check the instance's public IP
terraform output instance-public-ip
# SSH into the instance
ssh -i ~/.ssh/oci_free ubuntu@<public_ip>
# Or this command in one
ssh -i ~/.ssh/oci_free ubuntu@$(terraform output -raw instance-public-ip)
```

<!-- BEGIN_TF_DOCS -->
## Requirements

| Name | Version |
|------|---------|
| <a name="requirement_terraform"></a> [terraform](#requirement\_terraform) | >= 1.13.0 |
| <a name="requirement_oci"></a> [oci](#requirement\_oci) | ~> 7.15.0 |
| <a name="requirement_tls"></a> [tls](#requirement\_tls) | ~> 4.1.0 |

## Providers

| Name | Version |
|------|---------|
| <a name="provider_oci"></a> [oci](#provider\_oci) | ~> 7.15.0 |
| <a name="provider_tls"></a> [tls](#provider\_tls) | ~> 4.1.0 |

## Modules

No modules.

## Resources

| Name | Type |
|------|------|
| [oci_core_instance.free_instance](https://registry.terraform.io/providers/oracle/oci/latest/docs/resources/core_instance) | resource |
| [oci_core_internet_gateway.free_internet_gateway](https://registry.terraform.io/providers/oracle/oci/latest/docs/resources/core_internet_gateway) | resource |
| [oci_core_route_table.free_route_table](https://registry.terraform.io/providers/oracle/oci/latest/docs/resources/core_route_table) | resource |
| [oci_core_security_list.free_security_list](https://registry.terraform.io/providers/oracle/oci/latest/docs/resources/core_security_list) | resource |
| [oci_core_subnet.free_subnet](https://registry.terraform.io/providers/oracle/oci/latest/docs/resources/core_subnet) | resource |
| [oci_core_vcn.free_vcn](https://registry.terraform.io/providers/oracle/oci/latest/docs/resources/core_vcn) | resource |
| [oci_identity_compartment.free_compartment](https://registry.terraform.io/providers/oracle/oci/latest/docs/resources/identity_compartment) | resource |
| [tls_private_key.instance_ssh_key](https://registry.terraform.io/providers/hashicorp/tls/latest/docs/resources/private_key) | resource |
| [oci_core_images.instance_images](https://registry.terraform.io/providers/oracle/oci/latest/docs/data-sources/core_images) | data source |
| [oci_identity_availability_domains.ads](https://registry.terraform.io/providers/oracle/oci/latest/docs/data-sources/identity_availability_domains) | data source |

## Inputs

| Name | Description | Type | Default | Required |
|------|-------------|------|---------|:--------:|
| <a name="input_fingerprint"></a> [fingerprint](#input\_fingerprint) | The fingerprint for the user's RSA key. This can be found in user settings in the Oracle Cloud Infrastructure console. Required if auth is set to 'ApiKey', ignored otherwise. | `string` | n/a | yes |
| <a name="input_image_operating_system"></a> [image\_operating\_system](#input\_image\_operating\_system) | The image's operating system. | `string` | `"Canonical Ubuntu"` | no |
| <a name="input_image_operating_system_version"></a> [image\_operating\_system\_version](#input\_image\_operating\_system\_version) | The image's operating system version. | `string` | `"24.04"` | no |
| <a name="input_instance_boot_volume_size"></a> [instance\_boot\_volume\_size](#input\_instance\_boot\_volume\_size) | The size of the boot volume in GBs. Minimum value is 50 GB and maximum value is 32,768 GB (32 TB). | `number` | `200` | no |
| <a name="input_instance_hostname"></a> [instance\_hostname](#input\_instance\_hostname) | A user-friendly name. Does not have to be unique, and it's changeable. Avoid entering confidential information. | `string` | `"free"` | no |
| <a name="input_instance_memory"></a> [instance\_memory](#input\_instance\_memory) | The total amount of memory available to the instance, in gigabytes. | `number` | `24` | no |
| <a name="input_instance_ocpus"></a> [instance\_ocpus](#input\_instance\_ocpus) | The total number of OCPUs available to the instance. | `number` | `4` | no |
| <a name="input_instance_public_key_path"></a> [instance\_public\_key\_path](#input\_instance\_public\_key\_path) | Public SSH key to be included in the ~/.ssh/authorized\_keys file for the default user on the instance. | `string` | `""` | no |
| <a name="input_instance_shape"></a> [instance\_shape](#input\_instance\_shape) | The shape of an instance. The shape determines the number of CPUs, amount of memory, and other resources allocated to the instance. | `string` | `"VM.Standard.A1.Flex"` | no |
| <a name="input_private_key_path"></a> [private\_key\_path](#input\_private\_key\_path) | The path to the user's PEM formatted private key. A private\_key or a private\_key\_path must be provided if auth is set to 'ApiKey', ignored otherwise. | `string` | n/a | yes |
| <a name="input_region"></a> [region](#input\_region) | The region for API connections (e.g. us-ashburn-1). | `string` | n/a | yes |
| <a name="input_namespace"></a> [namespace](#input\_namespace) | The namespace of the OCI Object Storage. | `string` | n/a | yes |
| <a name="input_tenancy_ocid"></a> [tenancy\_ocid](#input\_tenancy\_ocid) | The tenancy OCID for a user. The tenancy OCID can be found at the bottom of user settings in the Oracle Cloud Infrastructure console. Required if auth is set to 'ApiKey', ignored otherwise. | `string` | n/a | yes |
| <a name="input_user_ocid"></a> [user\_ocid](#input\_user\_ocid) | The user OCID. This can be found in user settings in the Oracle Cloud Infrastructure console. Required if auth is set to 'ApiKey', ignored otherwise. | `string` | n/a | yes |

## Outputs

| Name | Description |
|------|-------------|
| <a name="output_compartment-id"></a> [compartment-id](#output\_compartment-id) | The OCID of the compartment. |
| <a name="output_compartment-name"></a> [compartment-name](#output\_compartment-name) | The name you assign to the compartment during creation. The name must be unique across all compartments in the parent. |
| <a name="output_instance-name"></a> [instance-name](#output\_instance-name) | A user-friendly name. Does not have to be unique, and it's changeable. |
| <a name="output_instance-public-ip"></a> [instance-public-ip](#output\_instance-public-ip) | The public IP address of instance VNIC (if enabled). |
| <a name="output_instance-region"></a> [instance-region](#output\_instance-region) | The region that contains the availability domain the instance is running in. |
| <a name="output_instance-shape"></a> [instance-shape](#output\_instance-shape) | The shape of the instance. The shape determines the number of CPUs and the amount of memory allocated to the instance. |
| <a name="output_instance-state"></a> [instance-state](#output\_instance-state) | The current state of the instance. |
<!-- END_TF_DOCS -->
