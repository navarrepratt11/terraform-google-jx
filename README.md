# Jenkins X GKE Module

![Terraform Version](https://img.shields.io/badge/tf-%3E%3D0.12.0-blue.svg)

__NOTE:__ While the required minimum Terraform version is 0.12.0, automated CI tests are performed with 0.13 only.  The only expected
compatibility issues to be aware of are around provider requirements.  For more information see [here](https://www.terraform.io/docs/configuration/provider-requirements.html#v0-12-compatible-provider-requirements)

---

This repo contains a [Terraform](https://www.terraform.io/) module for provisioning a Kubernetes cluster for [Jenkins X](https://jenkins-x.io/) on [Google Cloud](https://cloud.google.com/).

<!-- TOC -->

- [What is a Terraform module](#what-is-a-terraform-module)
- [How do you use this module](#how-do-you-use-this-module)
    - [Prerequisites](#prerequisites)
    - [Cluster provisioning](#cluster-provisioning)
        - [Inputs](#inputs)
        - [Outputs](#outputs)
    - [Running `jx boot`](#running-jx-boot)
    - [Using a custom domain](#using-a-custom-domain)
    - [Production cluster considerations](#production-cluster-considerations)
    - [Configuring a Terraform backend](#configuring-a-terraform-backend)
- [FAQ](#faq)
    - [How do I get the latest version of the terraform-google-jx module](#how-do-i-get-the-latest-version-of-the-terraform-google-jx-module)
    - [How to I specify a specific google provider version](#how-to-i-specify-a-specific-google-provider-version)
    - [Why do I need Application Default Credentials](#why-do-i-need-application-default-credentials)
- [Development](#development)
    - [Releasing](#releasing)
- [How do I contribute](#how-do-i-contribute)

<!-- /TOC -->

## What is a Terraform module

A Terraform "module" refers to a self-contained package of Terraform configurations that are managed as a group.
For more information around modules refer to the Terraform [documentation](https://www.terraform.io/docs/modules/index.html).

## How do you use this module

### Prerequisites

To make use of this module, you need a Google Cloud project.
Instructions on how to setup such a project can be found in the  [Google Cloud Installation and Setup](https://cloud.google.com/deployment-manager/docs/step-by-step-guide/installation-and-setup) guide.
You need your Google Cloud project id as an input variable for using this module.

You also need to install the Cloud SDK, in particular `gcloud`.
You find instructions on how to install and authenticate in the [Google Cloud Installation and Setup](https://cloud.google.com/deployment-manager/docs/step-by-step-guide/installation-and-setup) guide as well.

Once you have `gcloud` installed, you need to create [Application Default Credentials](https://cloud.google.com/sdk/gcloud/reference/auth/application-default/login) by running:

```bash
gcloud auth application-default login
```

Alternatively, you can export the environment variable _GOOGLE_APPLICATION_CREDENTIALS_ referencing the path to a Google Cloud [service account key file](https://cloud.google.com/iam/docs/creating-managing-service-account-keys).

Last but not least, ensure you have the following binaries installed:

- `gcloud`
- `kubectl` ~> 1.14.0
    - `kubectl` comes bundled with the Cloud SDK
- `terraform` ~> 0.12.0
    - Terraform installation instruction can be found [here](https://learn.hashicorp.com/terraform/getting-started/install)

### Cluster provisioning

A default Jenkins X ready cluster can be provisioned by creating a file _main.tf_ in an empty directory with the following content:

```terraform
module "jx" {
  source  = "jenkins-x/jx/google"

  gcp_project = "<my-gcp-project-id>"
}

output "jx_requirements" {
  value = module.jx.jx_requirements
}  
```

You can then apply this Terraform configuration via:

```bash
terraform init
terraform apply
```

This creates a cluster within the specified Google Cloud project with all possible configuration options defaulted.

:warning: **Note**: This example is for getting up and running quickly.
It is not intended for a production cluster.
Refer to [Production cluster considerations](#production-cluster-considerations) for things to consider when creating a production cluster.

On completion of `terraform apply` a _jx\_requirements_ output is available which can be used as input to `jx boot`.
Refer to [Running `jx boot`](#running-jx-boot) for more information.

In the default configuration, no custom domain is used.
DNS resolution occurs via [nip.io](https://nip.io/).
For more information on how to configure and use a custom domain, refer to [Using a custom domain](#using-a-custom-domain).

If you just want to experiment with Jenkins X, you can set `force_destroy` to `true`.
This allows you to remove all generated resources when running `terraform destroy`, including any generated buckets including their content.

The following two paragraphs provide the full list of configuration and output variables of this Terraform module.

## Requirements

| Name | Version |
|------|---------|
| <a name="requirement_terraform"></a> [terraform](#requirement\_terraform) | >= 0.12.0, < 0.15 |
| <a name="requirement_google"></a> [google](#requirement\_google) | >= 3.46.0 |
| <a name="requirement_google-beta"></a> [google-beta](#requirement\_google-beta) | >= 3.46.0 |
| <a name="requirement_helm"></a> [helm](#requirement\_helm) | ~>1.3.0 |
| <a name="requirement_kubernetes"></a> [kubernetes](#requirement\_kubernetes) | ~>1.11.0 |
| <a name="requirement_local"></a> [local](#requirement\_local) | >= 1.2.0 |
| <a name="requirement_null"></a> [null](#requirement\_null) | >= 2.1.0 |
| <a name="requirement_random"></a> [random](#requirement\_random) | >= 2.2.0 |
| <a name="requirement_template"></a> [template](#requirement\_template) | >= 2.1.0 |

## Providers

| Name | Version |
|------|---------|
| <a name="provider_google"></a> [google](#provider\_google) | 3.83.0 |
| <a name="provider_random"></a> [random](#provider\_random) | 3.1.0 |

## Modules

| Name | Source | Version |
|------|--------|---------|
| <a name="module_backup"></a> [backup](#module\_backup) | ./modules/backup | n/a |
| <a name="module_cluster"></a> [cluster](#module\_cluster) | ./modules/cluster | n/a |
| <a name="module_dns"></a> [dns](#module\_dns) | ./modules/dns | n/a |
| <a name="module_gsm"></a> [gsm](#module\_gsm) | ./modules/gsm | n/a |
| <a name="module_jx-boot"></a> [jx-boot](#module\_jx-boot) | ./modules/jx-boot | n/a |
| <a name="module_vault"></a> [vault](#module\_vault) | ./modules/vault | n/a |

## Resources

| Name | Type |
|------|------|
| [google_project_service.cloudbuild_api](https://registry.terraform.io/providers/hashicorp/google/latest/docs/resources/project_service) | resource |
| [google_project_service.cloudresourcemanager_api](https://registry.terraform.io/providers/hashicorp/google/latest/docs/resources/project_service) | resource |
| [google_project_service.compute_api](https://registry.terraform.io/providers/hashicorp/google/latest/docs/resources/project_service) | resource |
| [google_project_service.container_api](https://registry.terraform.io/providers/hashicorp/google/latest/docs/resources/project_service) | resource |
| [google_project_service.containeranalysis_api](https://registry.terraform.io/providers/hashicorp/google/latest/docs/resources/project_service) | resource |
| [google_project_service.containerregistry_api](https://registry.terraform.io/providers/hashicorp/google/latest/docs/resources/project_service) | resource |
| [google_project_service.iam_api](https://registry.terraform.io/providers/hashicorp/google/latest/docs/resources/project_service) | resource |
| [google_project_service.serviceusage_api](https://registry.terraform.io/providers/hashicorp/google/latest/docs/resources/project_service) | resource |
| [random_id.random](https://registry.terraform.io/providers/hashicorp/random/latest/docs/resources/id) | resource |
| [random_pet.current](https://registry.terraform.io/providers/hashicorp/random/latest/docs/resources/pet) | resource |
| [google_client_config.default](https://registry.terraform.io/providers/hashicorp/google/latest/docs/data-sources/client_config) | data source |

## Inputs

| Name | Description | Type | Default | Required |
|------|-------------|------|---------|:--------:|
| <a name="input_apex_domain"></a> [apex\_domain](#input\_apex\_domain) | The parent / apex domain to be used for the cluster | `string` | `""` | no |
| <a name="input_apex_domain_gcp_project"></a> [apex\_domain\_gcp\_project](#input\_apex\_domain\_gcp\_project) | The GCP project the apex domain is managed by, used to write recordsets for a subdomain if set.  Defaults to current project. | `string` | `""` | no |
| <a name="input_apex_domain_integration_enabled"></a> [apex\_domain\_integration\_enabled](#input\_apex\_domain\_integration\_enabled) | Flag that when set attempts to create delegation records in apex domain to point to domain created by this module | `bool` | `true` | no |
| <a name="input_bucket_location"></a> [bucket\_location](#input\_bucket\_location) | Bucket location for storage | `string` | `"US"` | no |
| <a name="input_cluster_ipv4_cidr_block"></a> [cluster\_ipv4\_cidr\_block](#input\_cluster\_ipv4\_cidr\_block) | Pod secondary CIDR range | `string` | n/a | yes |
| <a name="input_cluster_location"></a> [cluster\_location](#input\_cluster\_location) | The location (region or zone) in which the cluster master will be created. If you specify a zone (such as us-central1-a), the cluster will be a zonal cluster with a single cluster master. If you specify a region (such as us-west1), the cluster will be a regional cluster with multiple masters spread across zones in the region | `string` | `"us-central1-a"` | no |
| <a name="input_cluster_name"></a> [cluster\_name](#input\_cluster\_name) | Name of the Kubernetes cluster to create | `string` | `""` | no |
| <a name="input_cluster_network"></a> [cluster\_network](#input\_cluster\_network) | The name of the network (VPC) to which the cluster is connected | `string` | `"default"` | no |
| <a name="input_cluster_subnetwork"></a> [cluster\_subnetwork](#input\_cluster\_subnetwork) | The name of the subnetwork to which the cluster is connected. Leave blank when using the 'default' vpc to generate a subnet for your cluster | `string` | `""` | no |
| <a name="input_create_ui_sa"></a> [create\_ui\_sa](#input\_create\_ui\_sa) | Whether the service accounts for the UI should be created | `bool` | `true` | no |
| <a name="input_dev_env_approvers"></a> [dev\_env\_approvers](#input\_dev\_env\_approvers) | List of git users allowed to approve pull request for dev enviornment repository | `list(string)` | `[]` | no |
| <a name="input_enable_backup"></a> [enable\_backup](#input\_enable\_backup) | Whether or not Velero backups should be enabled | `bool` | `false` | no |
| <a name="input_force_destroy"></a> [force\_destroy](#input\_force\_destroy) | Flag to determine whether storage buckets get forcefully destroyed | `bool` | `false` | no |
| <a name="input_gcp_project"></a> [gcp\_project](#input\_gcp\_project) | The name of the GCP project to use | `string` | n/a | yes |
| <a name="input_git_owner_requirement_repos"></a> [git\_owner\_requirement\_repos](#input\_git\_owner\_requirement\_repos) | The git id of the owner for the requirement repositories | `string` | `""` | no |
| <a name="input_gsm"></a> [gsm](#input\_gsm) | Enables Google Secrets Manager, not available with JX2 | `bool` | `false` | no |
| <a name="input_jenkins_x_namespace"></a> [jenkins\_x\_namespace](#input\_jenkins\_x\_namespace) | Kubernetes namespace to install Jenkins X in | `string` | `"jx"` | no |
| <a name="input_jx2"></a> [jx2](#input\_jx2) | Is a Jenkins X 2 install | `bool` | `true` | no |
| <a name="input_jx_bot_token"></a> [jx\_bot\_token](#input\_jx\_bot\_token) | Bot token used to interact with the Jenkins X cluster git repository | `string` | `""` | no |
| <a name="input_jx_bot_username"></a> [jx\_bot\_username](#input\_jx\_bot\_username) | Bot username used to interact with the Jenkins X cluster git repository | `string` | `""` | no |
| <a name="input_jx_git_operator_version"></a> [jx\_git\_operator\_version](#input\_jx\_git\_operator\_version) | The jx-git-operator helm chart version | `string` | `"0.0.192"` | no |
| <a name="input_jx_git_url"></a> [jx\_git\_url](#input\_jx\_git\_url) | URL for the Jenins X cluster git repository | `string` | `""` | no |
| <a name="input_kuberhealthy"></a> [kuberhealthy](#input\_kuberhealthy) | Enables Kuberhealthy helm installation | `bool` | `true` | no |
| <a name="input_lets_encrypt_production"></a> [lets\_encrypt\_production](#input\_lets\_encrypt\_production) | Flag to determine wether or not to use the Let's Encrypt production server. | `bool` | `true` | no |
| <a name="input_master_ipv4_cidr_block"></a> [master\_ipv4\_cidr\_block](#input\_master\_ipv4\_cidr\_block) | Control plane IP range | `string` | n/a | yes |
| <a name="input_max_node_count"></a> [max\_node\_count](#input\_max\_node\_count) | Maximum number of cluster nodes | `number` | `5` | no |
| <a name="input_min_node_count"></a> [min\_node\_count](#input\_min\_node\_count) | Minimum number of cluster nodes | `number` | `3` | no |
| <a name="input_node_disk_size"></a> [node\_disk\_size](#input\_node\_disk\_size) | Node disk size in GB | `string` | `"100"` | no |
| <a name="input_node_disk_type"></a> [node\_disk\_type](#input\_node\_disk\_type) | Node disk type, either pd-standard or pd-ssd | `string` | `"pd-standard"` | no |
| <a name="input_node_machine_type"></a> [node\_machine\_type](#input\_node\_machine\_type) | Node type for the Kubernetes cluster | `string` | `"n1-standard-2"` | no |
| <a name="input_node_preemptible"></a> [node\_preemptible](#input\_node\_preemptible) | Use preemptible nodes | `bool` | `false` | no |
| <a name="input_parent_domain"></a> [parent\_domain](#input\_parent\_domain) | **Deprecated** Please use apex\_domain variable instead.r | `string` | `""` | no |
| <a name="input_parent_domain_gcp_project"></a> [parent\_domain\_gcp\_project](#input\_parent\_domain\_gcp\_project) | **Deprecated** Please use apex\_domain\_gcp\_project variable instead. | `string` | `""` | no |
| <a name="input_release_channel"></a> [release\_channel](#input\_release\_channel) | The GKE release channel to subscribe to.  See https://cloud.google.com/kubernetes-engine/docs/concepts/release-channels | `string` | `"REGULAR"` | no |
| <a name="input_resource_labels"></a> [resource\_labels](#input\_resource\_labels) | Set of labels to be applied to the cluster | `map` | `{}` | no |
| <a name="input_services_ipv4_cidr_block"></a> [services\_ipv4\_cidr\_block](#input\_services\_ipv4\_cidr\_block) | Services secondary CIDR range | `string` | n/a | yes |
| <a name="input_subdomain"></a> [subdomain](#input\_subdomain) | Optional sub domain for the installation | `string` | `""` | no |
| <a name="input_tls_email"></a> [tls\_email](#input\_tls\_email) | Email used by Let's Encrypt. Required for TLS when apex\_domain is specified | `string` | `""` | no |
| <a name="input_uniform_bucket_level_access"></a> [uniform\_bucket\_level\_access](#input\_uniform\_bucket\_level\_access) | Boolean for setting this option when creating the necessary storage buckets | `bool` | `false` | no |
| <a name="input_vault_url"></a> [vault\_url](#input\_vault\_url) | URL to an external Vault instance in case Jenkins X shall not create its own system Vault | `string` | `""` | no |
| <a name="input_velero_namespace"></a> [velero\_namespace](#input\_velero\_namespace) | Kubernetes namespace for Velero | `string` | `"velero"` | no |
| <a name="input_velero_schedule"></a> [velero\_schedule](#input\_velero\_schedule) | The Velero backup schedule in cron notation to be set in the Velero Schedule CRD (see [default-backup.yaml](https://github.com/jenkins-x/jenkins-x-boot-config/blob/master/systems/velero-backups/templates/default-backup.yaml)) | `string` | `"0 * * * *"` | no |
| <a name="input_velero_ttl"></a> [velero\_ttl](#input\_velero\_ttl) | The the lifetime of a velero backup to be set in the Velero Schedule CRD (see [default-backup.yaml](https://github.com/jenkins-x/jenkins-x-boot-config/blob/master/systems/velero-backups/templates/default-backup)) | `string` | `"720h0m0s"` | no |
| <a name="input_version_stream_ref"></a> [version\_stream\_ref](#input\_version\_stream\_ref) | The git ref for version stream to use when booting Jenkins X. See https://jenkins-x.io/docs/concepts/version-stream/ | `string` | `"master"` | no |
| <a name="input_version_stream_url"></a> [version\_stream\_url](#input\_version\_stream\_url) | The URL for the version stream to use when booting Jenkins X. See https://jenkins-x.io/docs/concepts/version-stream/ | `string` | `"https://github.com/jenkins-x/jenkins-x-versions.git"` | no |
| <a name="input_webhook"></a> [webhook](#input\_webhook) | Jenkins X webhook handler for git provider | `string` | `"lighthouse"` | no |
| <a name="input_zone"></a> [zone](#input\_zone) | Zone in which to create the cluster (deprecated, use cluster\_location instead) | `string` | `""` | no |

## Outputs

| Name | Description |
|------|-------------|
| <a name="output_backup_bucket_url"></a> [backup\_bucket\_url](#output\_backup\_bucket\_url) | The URL to the bucket for backup storage |
| <a name="output_cluster_location"></a> [cluster\_location](#output\_cluster\_location) | The location of the created Kubernetes cluster |
| <a name="output_cluster_name"></a> [cluster\_name](#output\_cluster\_name) | The name of the created Kubernetes cluster |
| <a name="output_connect"></a> [connect](#output\_connect) | The cluster connection string to use once Terraform apply finishes |
| <a name="output_externaldns_dns_name"></a> [externaldns\_dns\_name](#output\_externaldns\_dns\_name) | ExternalDNS name |
| <a name="output_externaldns_ns"></a> [externaldns\_ns](#output\_externaldns\_ns) | ExternalDNS nameservers |
| <a name="output_gcp_project"></a> [gcp\_project](#output\_gcp\_project) | The GCP project in which the resources got created |
| <a name="output_jx_requirements"></a> [jx\_requirements](#output\_jx\_requirements) | The jx-requirements rendered output |
| <a name="output_log_storage_url"></a> [log\_storage\_url](#output\_log\_storage\_url) | The URL to the bucket for log storage |
| <a name="output_report_storage_url"></a> [report\_storage\_url](#output\_report\_storage\_url) | The URL to the bucket for report storage |
| <a name="output_repository_storage_url"></a> [repository\_storage\_url](#output\_repository\_storage\_url) | The URL to the bucket for artifact storage |
| <a name="output_tekton_sa_email"></a> [tekton\_sa\_email](#output\_tekton\_sa\_email) | The Tekton service account email address, useful to provide further IAM bindings |
| <a name="output_tekton_sa_name"></a> [tekton\_sa\_name](#output\_tekton\_sa\_name) | The Tekton service account name, useful to provide further IAM bindings |
| <a name="output_vault_bucket_url"></a> [vault\_bucket\_url](#output\_vault\_bucket\_url) | The URL to the bucket for secret storage |

### Running `jx boot`

A terraform output (_jx\_requirements_) is available after applying this Terraform module.

```sh
terraform output jx_requirements
```

This `jx_requirements` output can be used as input to [Jenkins X Boot](https://jenkins-x.io/docs/getting-started/setup/boot/) which is responsible for installing all the required Jenkins X components into the cluster created by this module.

![Jenkins X Installation/Update Flow](./images/terraform_google_jx.png)

:warning: **Note**: The generated _jx-requirements_ is only used for the first run of `jx boot`.
During this first run of `jx boot` a git repository containing the source code for Jenkins X Boot is created.
This (_new_) repository contains a _jx-requirements.yml_ (_which is now ahead of the jx-requirements output from terraform_) used by successive runs of `jx boot`.

Execute:

```sh
terraform output jx_requirements > <some_empty_dir>/jx-requirements.yml
# jenkins-x creates the environment repository directory localy before pushing to the Git server of choice
cd <some_empty_dir>
jx boot --requirements jx-requirements.yml
```

You are prompted for any further required configuration.
The number of prompts depends on how much you have [pre-configured](#inputs) via your Terraform variables.

### Using a custom domain

If you want to use a custom domain with your Jenkins X installation, you need to provide values for the [variables](#inputs) _apex\_domain_ and _tls\_email_.
_apex\_domain_ is the fully qualified domain name you want to use and _tls\_email_ is the email address you want to use for issuing Let's Encrypt TLS certificates.

Before you apply the Terraform configuration, you also need to create a [Cloud DNS managed zone](https://cloud.google.com/dns/zones), with the DNS name in the managed zone matching your custom domain name, for example in the case of _example.jenkins-x.rocks_ as domain:

![Creating a Managed Zone](./images/create_managed_zone.png)

When creating the managed zone, a set of DNS servers get created which you need to specify in the DNS settings of your DNS registrar.

![DNS settings of Managed Zone](./images/managed_zone_details.png)

It is essential that before you run `jx boot`, your DNS servers settings are propagated, and your domain resolves.
You can use [DNS checker](https://dnschecker.org/) to check whether your domain settings have propagated.

When a custom domain is provided, Jenkins X uses [ExternalDNS](https://github.com/kubernetes-sigs/external-dns) together with [cert-manager](https://github.com/jetstack/cert-manager) to create A record entries in your managed zone for the various exposed applications.

If _apex_domain_ id not set, your cluster will use [nip.io](https://nip.io/) in order to create publicly resolvable URLs of the form ht<span>tp://\<app-name\>-\<environment-name\>.\<cluster-ip\>.nip.io.

### Production cluster considerations

The configuration as seen in [Cluster provisioning](#cluster-provisioning) is not suited for creating and maintaining a production Jenkins X cluster.
The following is a list of considerations for a production usecase.

- Specify the version attribute of the module, for example:

    ```terraform
    module "jx" {
      source  = "jenkins-x/jx/google"
      version = "1.2.4"
      # insert your configuration
    }

   output "jx_requirements" {
     value = module.jx.jx_requirements
   }
   ```

  Specifying the version ensures that you are using a fixed version and that version upgrades cannot occur unintented.

- Keep the Terraform configuration under version control,  by creating a dedicated repository for your cluster configuration or by adding it to an already existing infrastructure repository.

- Setup a Terraform backend to securely store and share the state of your cluster. For more information refer to [Configuring a Terraform backend](#configuring-a-terraform-backend).

### Configuring a Terraform backend

A "[backend](https://www.terraform.io/docs/backends/index.html)" in Terraform determines how state is loaded and how an operation such as _apply_ is executed.
By default, Terraform uses the _local_ backend which keeps the state of the created resources on the local file system.
This is problematic since sensitive information will be stored on disk and it is not possible to share state across a team.
When working with Google Cloud a good choice for your Terraform backend is the [_gcs_ backend](https://www.terraform.io/docs/backends/types/gcs.html)  which stores the Terraform state in a Google Cloud Storage bucket.
The [examples](./examples) directory of this repository contains configuration examples for using the gcs backed with and without optionally configured customer supplied encryption key.

To use the gcs backend you will need to create the bucket upfront.
You can use `gsutil` to create the bucket:

```sh
gsutil mb gs://<my-bucket-name>/
```

It is also recommended to enable versioning on the bucket as an additional safety net in case of state corruption.

```sh
gsutil versioning set on gs://<my-bucket-name>
```

You can verify whether a bucket has versioning enabled via:

```sh
gsutil versioning get gs://<my-bucket-name>
```

## FAQ

### How do I get the latest version of the terraform-google-jx module

```sh
terraform init -upgrade
```

### How to I specify a specific google provider version

```yaml
provider "google" {
  version = "~> 2.12.0"
  project = var.gcp_project
}

provider "google-beta" {
  version = "~> 2.12.0"
  project = var.gcp_project
}
```

### Why do I need Application Default Credentials

The recommended way to authenticate to the Google Cloud API is by using a [service account](https://cloud.google.com/docs/authentication/getting-started).
This allows for authentication regardless of where your code runs.
This Terraform module expects authentication via a service account key.
You can either specify the path to this key directly using the _GOOGLE_APPLICATION_CREDENTIALS_ environment variable or you can run `gcloud auth application-default login`.
In the latter case `gcloud` obtains user access credentials via a web flow and puts them in the well-known location for Application Default Credentials (ADC), usually _~/.config/gcloud/application_default_credentials.json_.

## Development

### Releasing

At the moment there is no release pipeline defined in [jenkins-x.yml](./jenkins-x.yml).
A Terraform release does not require building an artifact, only a tag needs to be created and pushed.
To make this task easier and there is a helper script `release.sh` which simplifies this process and creates the changelog as well:

```sh
./scripts/release.sh
```

This can be executed on demand whenever a release is required.
For the script to work the envrionment variable _$GH_TOKEN_ must be exported and reference a valid GitHub API token.

## How do I contribute

Contributions are very welcome! Check out the [Contribution Guidelines](./CONTRIBUTING.md) for instructions.
