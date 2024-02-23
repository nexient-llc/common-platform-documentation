# Git Repository Naming Scheme

Names for our Git repositories are standardized to provide a predictable format for easy lookup and retrieval.

IaC Repositories should follow the following general format:

> tool-provider-type-resource_name

## Separators

We use hyphens (`-`) to separate fields in a repository's name. Only underscores (`_`) are used within a field to separate words, and any compound value for a **type** or **tool** should be expressed in `snake_case`, that is, lowercase words separated by underscores.

## Tool

**Tool** is the name of the IaC or provisioning tool with which this repository is intended to be used. Prefer a shortened version of the name where possible, e.g. `tf` instead of Terraform.

Some valid tools are:

- `lcaf` for Launch Common Automation Framework platform concerns
- `tf` for Terraform modules and providers
- `tg` for Terragrunt modules
- `helm` for Helm charts

## Provider

**Provider** is the name of the cloud/vendor for which this repository is intended.

Some valid providers are:

`aws` for Amazon Web Services Cloud
`azurerm` for Microsoft Azure Cloud
`gcp` for Google Cloud
`oci` for Oracle Cloud
`sumologic` for the SumoLogic vendor

For modules that include resources that span a cloud and a vendor, like our combined AWS/SumoLogic module that orchestrates resources in both, prefer the name of the cloud/hyperscaler for the **provider** value and relegate the vendor to the **resource_name** section, like so:

> tf-aws-module_reference-sumologic_observability

## Type

The **type** is a classification of the module type for the **tool** in question and will vary depending upon the tool, including but not limited to the examples found below:

- Terraform
    - `module_primitive` - TF modules that implement a single cloud resource and are used as building blocks for higher-order collections and reference architectures
    - `module_library` - TF modules that provide a library of functions or provide a set of complex outputs from simpler inputs, e.g. resource_name, which takes parameters and produces standardized names for use with other modules. Modules of this type should not directly instantiate cloud resources.
    - `module_collection` - Formerly known as wrapper modules, these TF modules serve as a convenient logical grouping of primitive resources and library modules, e.g. a Cloudwatch collection module may be capable of creating log groups, log streams, and log subscription filters from a single entrypoint.
    - `module_reference` - TF modules that represent a fully vetted reference architecture that can be deployed as a unit in the field, e.g. an entire ECS platform setup including the ECS cluster, load balancing, service mesh, etc.
    - `provider` - We don't currently have any custom Terraform providers, but if we need them in the future, the **type** field must be set to `provider`.

- Helm
    - library_chart
    - deployment_chart
    - ???

## Resource Name

A **resource_name** is the specific type of cloud resource or reference architecture in question. Separators in the name are limited to underscore (`_`) characters. 

When naming modules that correspond directly to cloud resources, prefer naming them exactly how the **provider** references them. `s3_bucket` and `key_vault` would be two examples of cloud resources.

When naming reference architectures, get the name as close to the **provider**'s name as possible and add any further specifiers.

For example, we use `ecs_platform` and `ecs_appmesh_platform` as the **resource_name** for our reference architectures for ECS (without and with AppMesh enabled, respectively).


<!-- 

Matthew Dresden
  10:21 AM
Can you take a stab and documenting our tf module and tf provider naming standardin markdown
our helm charts later will follow a similar pattern
meet with Ben to confirm the propsed nameing standard
something like
<tool>-<provider>-<module_type>-<resource or ref platform>
<tf>-<azurerm|aws|gcp|etc>-<module-(primtive|library|collection|reference)>-<resource|platform name>
<tf>-<author>-<provider>
tf-launch-azurerm for example
helm-umbrella_chart-aks_platform

-->