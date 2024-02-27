# Infrastructure as Code Git Repository Naming Scheme

Names for our IaC Git repositories are standardized to provide a predictable format for easy lookup and retrieval. Repository names are case-insensitive and should always be normalized to lowercase and folded to ASCII characters.

IaC Repositories should follow the following general format:

> tool-provider-type-resource

## Separators

We use hyphens (`-`) to separate fields in a repository's name. Only underscores (`_`) are used within a field to separate words, and any compound value for a [type](#type) or [tool](#tool) should be expressed in `snake_case`, that is, lowercase words separated by underscores.

## Tool

**Tool** is the name of the IaC or provisioning tool with which this repository is intended to be used. Prefer a shortened version of the name where possible, e.g. `tf` instead of Terraform.

Some valid tools are:

- `tf` for Terraform modules and providers
- `tg` for Terragrunt modules
- `helm` for Helm charts

## Provider

**Provider** is the name of the cloud/vendor for which this repository is intended.

Some valid providers are:

- `aws` for Amazon Web Services Cloud
- `azurerm` for Microsoft Azure Cloud
- `gcp` for Google Cloud
- `oci` for Oracle Cloud
- `sumologic` for the SumoLogic vendor
- `launch` for agnostic modules that do not have an affinity to a particular cloud
- `k8s` for Helm charts designed to work with any Kubernetes provider (e.g. AKS, EKS, GKE)

For modules that include resources that span a cloud and a vendor, like our combined AWS/SumoLogic module that orchestrates resources in both, prefer the name of the cloud/hyperscaler for the [provider](#provider) value and relegate the vendor to the [resource](#resource) section, like so:

> tf-aws-module_reference-sumologic_observability

## Type

The **type** is a classification of the module type for the [tool](#tool) in question and will vary depending upon the tool, including but not limited to the examples found below:

- Terraform
    - `module_primitive` - TF modules that implement a single cloud resource and are used as building blocks for higher-order collections and reference architectures
    - `module_library` - TF modules that provide a library of functions or provide a set of complex outputs from simpler inputs, e.g. resource_name, which takes parameters and produces standardized names for use with other modules. Modules of this type should not directly instantiate cloud resources.
    - `module_collection` - Formerly known as wrapper modules, these TF modules serve as a convenient logical grouping of primitive resources and library modules, e.g. a Cloudwatch collection module may be capable of creating log groups, log streams, and log subscription filters from a single entrypoint.
    - `module_reference` - TF modules that represent a fully vetted reference architecture that can be deployed as a unit in the field, e.g. an entire ECS platform setup including the ECS cluster, load balancing, service mesh, etc.
    - `provider` - We don't currently have any custom Terraform providers, but if we need them in the future, the [type](#type) field must be set to `provider`.

- Helm
    - `chart_library` - Equivalent to a `module_library` in Terraform, a library chart cannot stand alone and does not deploy any resources directly, instead it provides snippets for other charts to reuse.
    - `chart_deployment` - A deployment chart roughly maps to a `module_primitive` in Terraform, it represents a single deployment type (i.e. deployments for long-running services are distinct from those that run batch jobs and would be separate `chart_deployment`s)
    - `chart_umbrella` - A "chart of charts," roughly equivalent to our `module_collection` or `module_reference` modules for Terraform. These charts include at least one `chart_deployment` and are a single entrypoint that represents a complete solution for a single application, service, or job.

## Resource

A **resource** is the specific type of cloud resource or reference architecture in question. Separators in the name are limited to underscore (`_`) characters. 

When naming modules that correspond directly to cloud resources, prefer naming them exactly how the [provider](#provider) references them. `s3_bucket` and `key_vault` would be two examples of cloud resources.

When naming reference architectures, get the name as close to the [provider](#provider)'s name as possible and add any further specifiers. For example, we use `ecs_platform` and `ecs_appmesh_platform` as the **resource** for our reference architectures for ECS (without and with AppMesh enabled, respectively).

# Examples in Practice

Below are some example names and explanations of what they should contain.

- `tf-aws-module_primitive-s3_bucket`

Represents a single Amazon S3 bucket with no other resources that is ready to be consumed by other Terraform modules.

- `tf-launch-module_library-resource_name`

This library module takes pure inputs (e.g. product name, region name, instance environment, instance resource, etc.) and produces outputs with consistent names from those inputs. Library modules never produce actual cloud infrastructure, they only transform their inputs into different outputs. Since this library is intended for use on any cloud, it uses `launch` as its [provider](#provider).

- `tf-aws-module_collection-activemq_broker`

This collection module around an ActiveMQ broker would include multiple resources and is designed to orchestrate the broker at a level higher than its primitive module. This collection might include the ability to create a load balancer in front of the MQ broker, support outputting load balancer logs to an S3 bucket, as well as include all the necessary policies to configure log shipping and access between the load balancer and the MQ broker. It may also include the `tf-aws-module_library-resource_name` library module mentioned above to set a consistent naming scheme for all its resources.

- `tf-aws-module_reference-sumologic_observability`

This reference module contains everything you would need to deploy a full observability integration to AWS and SumoLogic. There would be AWS components (wrapped up in a `module_collection`) as well as Sumo-specific components (wrapped in their own separate `module_collection`). Reference modules should contain all the necessary resources (either directly or through child `module_collection`s) to go from a blank slate (just an account) to a fully-functional system or application. Since this module has a specific cloud and vendor for which it was designed, the [provider](#provider) is set to the cloud (`aws`) and the [resource](#resource) includes the vendor's name and its purpose (`sumologic_observability`)

- `helm-k8s-chart_library-launch`

A library chart containing Launch's "standard library" of Helm snippets. Using this chart doesn't actually result in any resources being created or destroyed.

- `helm-k8s-chart_deployment-containerized_application`
- `helm-k8s-chart_deployment-batch_job`

Two examples of different deployments that might be accomplished through Helm. Both of these deployment charts may pull from the same library chart in order to build up their templates.

- `helm-k8s-chart_umbrella-my_application`

A helm chart that covers all Kubernetes deployment concerns for a single service, called `my_application`. The complexity of the application will determine the number of deployment charts that this umbrella chart pulls in; a simple application may be a single deployment chart, where a more complex application that includes several services and background jobs may pull from several deployment charts.
