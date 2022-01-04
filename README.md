# Terraform Kubernetes Engine Module

This module handles opinionated Google Cloud Platform Kubernetes Engine cluster creation and configuration with Node Pools, IP MASQ, Network Policy, etc. This particular submodule creates a [private cluster](https://cloud.google.com/kubernetes-engine/docs/how-to/private-clusters)Beta features are enabled in this submodule.
The resources/services/activations/deletions that this module will create/trigger are:
- Create a GKE cluster with the provided addons
- Create GKE Node Pool(s) with provided configuration and attach to cluster
- Replace the default kube-dns configmap if `stub_domains` are provided
- Activate network policy if `network_policy` is true
- Add `ip-masq-agent` configmap with provided `non_masquerade_cidrs` if `configure_ip_masq` is true

Sub modules are provided for creating private clusters, beta private clusters, and beta public clusters as well.  Beta sub modules allow for the use of various GKE beta features. See the modules directory for the various sub modules.

## Private Cluster Details
For details on configuring private clusters with this module, check the [troubleshooting guide](https://github.com/terraform-google-modules/terraform-google-kubernetes-engine/blob/master/docs/private_clusters.md).

## Compatibility

This module is meant for use with Terraform 0.12. If you haven't
[upgraded][terraform-0.12-upgrade] and need a Terraform
0.11.x-compatible version of this module, the last released version
intended for Terraform 0.11.x is [3.0.0].

## Usage
There are multiple examples included in the [examples](https://github.com/terraform-google-modules/terraform-google-kubernetes-engine/tree/master/examples) folder but simple usage is as follows:

```hcl
# google_client_config and kubernetes provider must be explicitly specified like the following.
data "google_client_config" "default" {}

provider "kubernetes" {
  load_config_file       = false
  host                   = "https://${module.gke.endpoint}"
  token                  = data.google_client_config.default.access_token
  cluster_ca_certificate = base64decode(module.gke.ca_certificate)
}

module "gke" {
  source                     = "terraform-google-modules/kubernetes-engine/google//modules/beta-autopilot-private-cluster"
  project_id                 = "<PROJECT ID>"
  name                       = "gke-test-1"
  region                     = "us-central1"
  zones                      = ["us-central1-a", "us-central1-b", "us-central1-f"]
  network                    = "vpc-01"
  subnetwork                 = "us-central1-01"
  ip_range_pods              = "us-central1-01-gke-01-pods"
  ip_range_services          = "us-central1-01-gke-01-services"
  horizontal_pod_autoscaling = true
  enable_private_endpoint    = true
  enable_private_nodes       = true
  master_ipv4_cidr_block     = "10.0.0.0/28"
  enable_autopilot           = true

}
```

<!-- do not understand what this is about -->
Then perform the following commands on the root folder:

- `terraform init` to get the plugins
- `terraform plan` to see the infrastructure plan
- `terraform apply` to apply the infrastructure build
- `terraform destroy` to destroy the built infrastructure

<!-- BEGINNING OF PRE-COMMIT-TERRAFORM DOCS HOOK -->
## Inputs

| Name | Description | Type | Default | Required |
|------|-------------|------|---------|:--------:|
| add\_cluster\_firewall\_rules | Create additional firewall rules | `bool` | `false` | no |
| add\_master\_webhook\_firewall\_rules | Create master\_webhook firewall rules for ports defined in `firewall_inbound_ports` | `bool` | `false` | no |
| add\_shadow\_firewall\_rules | Create GKE shadow firewall (the same as default firewall rules with firewall logs enabled). | `bool` | `false` | no |
| authenticator\_security\_group | The name of the RBAC security group for use with Google security groups in Kubernetes RBAC. Group name must be in format gke-security-groups@yourdomain.com | `string` | `null` | no |
| basic\_auth\_password | The password to be used with Basic Authentication. | `string` | `""` | no |
| basic\_auth\_username | The username to be used with Basic Authentication. An empty value will disable Basic Authentication, which is the recommended configuration. | `string` | `""` | no |
| cloudrun | (Beta) Enable CloudRun addon | `bool` | `false` | no |
| cloudrun\_load\_balancer\_type | (Beta) Configure the Cloud Run load balancer type. External by default. Set to `LOAD_BALANCER_TYPE_INTERNAL` to configure as an internal load balancer. | `string` | `""` | no |
| cluster\_autoscaling | Cluster autoscaling configuration. See [more details](https://cloud.google.com/kubernetes-engine/docs/reference/rest/v1beta1/projects.locations.clusters#clusterautoscaling) | <pre>object({<br>    enabled             = bool<br>    autoscaling_profile = string<br>    min_cpu_cores       = number<br>    max_cpu_cores       = number<br>    min_memory_gb       = number<br>    max_memory_gb       = number<br>  })</pre> | <pre>{<br>  "autoscaling_profile": "BALANCED",<br>  "enabled": false,<br>  "max_cpu_cores": 0,<br>  "max_memory_gb": 0,<br>  "min_cpu_cores": 0,<br>  "min_memory_gb": 0<br>}</pre> | no |
| cluster\_ipv4\_cidr | The IP address range of the kubernetes pods in this cluster. Default is an automatically assigned CIDR. | `any` | `null` | no |
| cluster\_resource\_labels | The GCE resource labels (a map of key/value pairs) to be applied to the cluster | `map(string)` | `{}` | no |
| cluster\_telemetry\_type | Available options include ENABLED, DISABLED, and SYSTEM\_ONLY | `string` | `null` | no |
| config\_connector | (Beta) Whether ConfigConnector is enabled for this cluster. | `bool` | `false` | no |
| configure\_ip\_masq | Enables the installation of ip masquerading, which is usually no longer required when using aliasied IP addresses. IP masquerading uses a kubectl call, so when you have a private cluster, you will need access to the API server. | `bool` | `false` | no |
| create\_service\_account | Defines if service account specified to run nodes should be created. | `bool` | `true` | no |
| database\_encryption | Application-layer Secrets Encryption settings. The object format is {state = string, key\_name = string}. Valid values of state are: "ENCRYPTED"; "DECRYPTED". key\_name is the name of a CloudKMS key. | `list(object({ state = string, key_name = string }))` | <pre>[<br>  {<br>    "key_name": "",<br>    "state": "DECRYPTED"<br>  }<br>]</pre> | no |
| datapath\_provider | The desired datapath provider for this cluster. By default, uses the IPTables-based kube-proxy implementation. | `string` | `"DATAPATH_PROVIDER_UNSPECIFIED"` | no |
| default\_max\_pods\_per\_node | The maximum number of pods to schedule per node | `number` | `110` | no |
| deploy\_using\_private\_endpoint | (Beta) A toggle for Terraform and kubectl to connect to the master's internal IP address during deployment. | `bool` | `false` | no |
| description | The description of the cluster | `string` | `""` | no |
| disable\_default\_snat | Whether to disable the default SNAT to support the private use of public IP addresses | `bool` | `false` | no |
| disable\_legacy\_metadata\_endpoints | Disable the /0.1/ and /v1beta1/ metadata server endpoints on the node. Changing this value will cause all node pools to be recreated. | `bool` | `true` | no |
| dns\_cache | (Beta) The status of the NodeLocal DNSCache addon. | `bool` | `true` | no |
| enable\_binary\_authorization | Enable BinAuthZ Admission controller | `bool` | `false` | no |
| enable\_intranode\_visibility | Whether Intra-node visibility is enabled for this cluster. This makes same node pod to pod traffic visible for VPC network | `bool` | `false` | no |
| enable\_kubernetes\_alpha | Whether to enable Kubernetes Alpha features for this cluster. Note that when this option is enabled, the cluster cannot be upgraded and will be automatically deleted after 30 days. | `bool` | `false` | no |
| enable\_network\_egress\_export | Whether to enable network egress metering for this cluster. If enabled, a daemonset will be created in the cluster to meter network egress traffic. | `bool` | `false` | no |
| enable\_pod\_security\_policy | enabled - Enable the PodSecurityPolicy controller for this cluster. If enabled, pods must be valid under a PodSecurityPolicy to be created. | `bool` | `false` | no |
| enable\_private\_endpoint | (Beta) Whether the master's internal IP address is used as the cluster endpoint | `bool` | `false` | no |
| enable\_private\_nodes | (Beta) Whether nodes have internal IP addresses only | `bool` | `false` | no |
| enable\_resource\_consumption\_export | Whether to enable resource consumption metering on this cluster. When enabled, a table will be created in the resource export BigQuery dataset to store resource consumption data. The resulting table can be joined with the resource usage table or with BigQuery billing export. | `bool` | `true` | no |
| enable\_shielded\_nodes | Enable Shielded Nodes features on all nodes in this cluster | `bool` | `true` | no |
| enable\_tpu | Enable Cloud TPU resources in the cluster. WARNING: changing this after cluster creation is destructive! | `bool` | `false` | no |
| enable\_vertical\_pod\_autoscaling | Vertical Pod Autoscaling automatically adjusts the resources of pods controlled by it | `bool` | `false` | no |
| firewall\_inbound\_ports | List of TCP ports for admission/webhook controllers. Either flag `add_master_webhook_firewall_rules` or `add_cluster_firewall_rules` (also adds egress rules) must be set to `true` for inbound-ports firewall rules to be applied. | `list(string)` | <pre>[<br>  "8443",<br>  "9443",<br>  "15017"<br>]</pre> | no |
| firewall\_priority | Priority rule for firewall rules | `number` | `1000` | no |
| gce\_pd\_csi\_driver | (Beta) Whether this cluster should enable the Google Compute Engine Persistent Disk Container Storage Interface (CSI) Driver. | `bool` | `true` | no |
| gcloud\_upgrade | Whether to upgrade gcloud at runtime | `bool` | `false` | no |
| grant\_registry\_access | Grants created cluster-specific service account storage.objectViewer role. | `bool` | `false` | no |
| horizontal\_pod\_autoscaling | Enable horizontal pod autoscaling addon | `bool` | `true` | no |
| http\_load\_balancing | Enable httpload balancer addon | `bool` | `true` | no |
| identity\_namespace | Workload Identity namespace. (Default value of `enabled` automatically sets project based namespace `[project_id].svc.id.goog`) | `string` | `"enabled"` | no |
| impersonate\_service\_account | An optional service account to impersonate for gcloud commands. If this service account is not specified, the module will use Application Default Credentials. | `string` | `""` | no |
| ip\_masq\_link\_local | Whether to masquerade traffic to the link-local prefix (169.254.0.0/16). | `bool` | `false` | no |
| ip\_masq\_resync\_interval | The interval at which the agent attempts to sync its ConfigMap file from the disk. | `string` | `"60s"` | no |
| ip\_range\_pods | The _name_ of the secondary subnet ip range to use for pods | `string` | n/a | yes |
| ip\_range\_services | The _name_ of the secondary subnet range to use for services | `string` | n/a | yes |
| issue\_client\_certificate | Issues a client certificate to authenticate to the cluster endpoint. To maximize the security of your cluster, leave this option disabled. Client certificates don't automatically rotate and aren't easily revocable. WARNING: changing this after cluster creation is destructive! | `bool` | `false` | no |
| istio | (Beta) Enable Istio addon | `bool` | `false` | no |
| istio\_auth | (Beta) The authentication type between services in Istio. | `string` | `"AUTH_MUTUAL_TLS"` | no |
| kalm\_config | (Beta) Whether KALM is enabled for this cluster. | `bool` | `false` | no |
| kubernetes\_version | The Kubernetes version of the masters. If set to 'latest' it will pull latest available version in the selected region. | `string` | `"latest"` | no |
| logging\_service | The logging service that the cluster should write logs to. Available options include logging.googleapis.com, logging.googleapis.com/kubernetes (beta), and none | `string` | `"logging.googleapis.com/kubernetes"` | no |
| maintenance\_end\_time | Time window specified for recurring maintenance operations in RFC3339 format | `string` | `""` | no |
| maintenance\_exclusions | List of maintenance exclusions. A cluster can have up to three | `list(object({ name = string, start_time = string, end_time = string }))` | `[]` | no |
| maintenance\_recurrence | Frequency of the recurring maintenance window in RFC5545 format. | `string` | `""` | no |
| maintenance\_start\_time | Time window specified for daily or recurring maintenance operations in RFC3339 format | `string` | `"05:00"` | no |
| master\_authorized\_networks | List of master authorized networks. If none are provided, disallow external access (except the cluster node IPs, which GKE automatically whitelists). | `list(object({ cidr_block = string, display_name = string }))` | `[]` | no |
| master\_global\_access\_enabled | (Beta) Whether the cluster master is accessible globally (from any region) or only within the same region as the private endpoint. | `bool` | `true` | no |
| master\_ipv4\_cidr\_block | (Beta) The IP range in CIDR notation to use for the hosted master network | `string` | `"10.0.0.0/28"` | no |
| monitoring\_service | The monitoring service that the cluster should write metrics to. Automatically send metrics from pods in the cluster to the Google Cloud Monitoring API. VM metrics will be collected by Google Compute Engine regardless of this setting Available options include monitoring.googleapis.com, monitoring.googleapis.com/kubernetes (beta) and none | `string` | `"monitoring.googleapis.com/kubernetes"` | no |
| name | The name of the cluster (required) | `string` | n/a | yes |
| network | The VPC network to host the cluster in (required) | `string` | n/a | yes |
| network\_policy | Enable network policy addon | `bool` | `false` | no |
| network\_policy\_provider | The network policy provider. | `string` | `"CALICO"` | no |
| network\_project\_id | The project ID of the shared VPC's host (for shared vpc support) | `string` | `""` | no |
| node\_metadata | Specifies how node metadata is exposed to the workload running on the node | `string` | `"GKE_METADATA_SERVER"` | no |
| non\_masquerade\_cidrs | List of strings in CIDR notation that specify the IP address ranges that do not use IP masquerading. | `list(string)` | <pre>[<br>  "10.0.0.0/8",<br>  "172.16.0.0/12",<br>  "192.168.0.0/16"<br>]</pre> | no |
| notification\_config\_topic | The desired Pub/Sub topic to which notifications will be sent by GKE. Format is projects/{project}/topics/{topic}. | `string` | `""` | no |
| project\_id | The project ID to host the cluster in (required) | `string` | n/a | yes |
| region | The region to host the cluster in (optional if zonal cluster / required if regional) | `string` | `null` | no |
| regional | Whether is a regional cluster (zonal cluster if set false. WARNING: changing this after cluster creation is destructive!) | `bool` | `true` | no |
| registry\_project\_ids | Projects holding Google Container Registries. If empty, we use the cluster project. If a service account is created and the `grant_registry_access` variable is set to `true`, the `storage.objectViewer` role is assigned on these projects. | `list(string)` | `[]` | no |
| release\_channel | The release channel of this cluster. Accepted values are `UNSPECIFIED`, `RAPID`, `REGULAR` and `STABLE`. Defaults to `UNSPECIFIED`. | `string` | `null` | no |
| resource\_usage\_export\_dataset\_id | The ID of a BigQuery Dataset for using BigQuery as the destination of resource usage export. | `string` | `""` | no |
| sandbox\_enabled | (Beta) Enable GKE Sandbox (Do not forget to set `image_type` = `COS_CONTAINERD` to use it). | `bool` | `false` | no |
| service\_account | The service account to run nodes as if not overridden in `node_pools`. The create\_service\_account variable default value (true) will cause a cluster-specific service account to be created. | `string` | `""` | no |
| shadow\_firewall\_rules\_priority | The firewall priority of GKE shadow firewall rules. The priority should be less than default firewall, which is 1000. | `number` | `999` | no |
| skip\_provisioners | Flag to skip all local-exec provisioners. It breaks `stub_domains` and `upstream_nameservers` variables functionality. | `bool` | `false` | no |
| stub\_domains | Map of stub domains and their resolvers to forward DNS queries for a certain domain to an external DNS server | `map(list(string))` | `{}` | no |
| subnetwork | The subnetwork to host the cluster in (required) | `string` | n/a | yes |
| upstream\_nameservers | If specified, the values replace the nameservers taken by default from the node’s /etc/resolv.conf | `list(string)` | `[]` | no |
| zones | The zones to host the cluster in (optional if regional cluster / required if zonal) | `list(string)` | `[]` | no |

## Outputs

| Name | Description |
|------|-------------|
| ca\_certificate | Cluster ca certificate (base64 encoded) |
| cloudrun\_enabled | Whether CloudRun enabled |
| dns\_cache\_enabled | Whether DNS Cache enabled |
| endpoint | Cluster endpoint |
| horizontal\_pod\_autoscaling\_enabled | Whether horizontal pod autoscaling enabled |
| http\_load\_balancing\_enabled | Whether http load balancing enabled |
| identity\_namespace | Workload Identity namespace |
| instance\_group\_urls | List of GKE generated instance groups |
| intranode\_visibility\_enabled | Whether intra-node visibility is enabled |
| istio\_enabled | Whether Istio is enabled |
| location | Cluster location (region if regional cluster, zone if zonal cluster) |
| logging\_service | Logging service used |
| master\_authorized\_networks\_config | Networks from which access to master is permitted |
| master\_ipv4\_cidr\_block | The IP range in CIDR notation used for the hosted master network |
| master\_version | Current master kubernetes version |
| min\_master\_version | Minimum master kubernetes version |
| monitoring\_service | Monitoring service used |
| name | Cluster name |
| peering\_name | The name of the peering between this cluster and the Google owned VPC. |
| pod\_security\_policy\_enabled | Whether pod security policy is enabled |
| region | Cluster region |
| release\_channel | The release channel of this cluster |
| service\_account | The service account to default running nodes as if not overridden in `node_pools`. |
| tpu\_ipv4\_cidr\_block | The IP range in CIDR notation used for the TPUs |
| type | Cluster type (regional / zonal) |
| vertical\_pod\_autoscaling\_enabled | Whether veritical pod autoscaling is enabled |
| zones | List of zones in which the cluster resides |

<!-- END OF PRE-COMMIT-TERRAFORM DOCS HOOK -->

## Requirements

Before this module can be used on a project, you must ensure that the following pre-requisites are fulfilled:

1. Terraform and kubectl are [installed](#software-dependencies) on the machine where Terraform is executed.
2. The Service Account you execute the module with has the right [permissions](#configure-a-service-account).
3. The Compute Engine and Kubernetes Engine APIs are [active](#enable-apis) on the project you will launch the cluster in.
4. If you are using a Shared VPC, the APIs must also be activated on the Shared VPC host project and your service account needs the proper permissions there.

The [project factory](https://github.com/terraform-google-modules/terraform-google-project-factory) can be used to provision projects with the correct APIs active and the necessary Shared VPC connections.

### Software Dependencies
#### Kubectl
- [kubectl](https://github.com/kubernetes/kubernetes/releases) 1.9.x
#### Terraform and Plugins
- [Terraform](https://www.terraform.io/downloads.html) 0.12
- [Terraform Provider for GCP Beta][terraform-provider-google-beta] v3.41
#### gcloud
Some submodules use the [terraform-google-gcloud](https://github.com/terraform-google-modules/terraform-google-gcloud) module. By default, this module assumes you already have gcloud installed in your $PATH.
See the [module](https://github.com/terraform-google-modules/terraform-google-gcloud#downloading) documentation for more information.

### Configure a Service Account
In order to execute this module you must have a Service Account with the
following project roles:
- roles/compute.viewer
- roles/compute.securityAdmin (only required if `add_cluster_firewall_rules` is set to `true`)
- roles/container.clusterAdmin
- roles/container.developer
- roles/iam.serviceAccountAdmin
- roles/iam.serviceAccountUser
- roles/resourcemanager.projectIamAdmin (only required if `service_account` is set to `create`)

Additionally, if `service_account` is set to `create` and `grant_registry_access` is requested, the service account requires the following role on the `registry_project_ids` projects:
- roles/resourcemanager.projectIamAdmin

### Enable APIs
In order to operate with the Service Account you must activate the following APIs on the project where the Service Account was created:

- Compute Engine API - compute.googleapis.com
- Kubernetes Engine API - container.googleapis.com

[terraform-provider-google-beta]: https://github.com/terraform-providers/terraform-provider-google-beta
[3.0.0]: https://registry.terraform.io/modules/terraform-google-modules/kubernetes-engine/google/3.0.0
[terraform-0.12-upgrade]: https://www.terraform.io/upgrade-guides/0-12.html
