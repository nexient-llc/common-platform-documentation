# Launch Terraform Strategy for Clouds, SaaS Products, and any other service where state needs to be managed

Terraform will be broadly used to manage the state of resources. This includes:

- Cloud resources
- SaaS resources
- Other services where state needs to be managed such as:
    - Kubernetes
    - Shared Databases
    - etc

## General Strategy

### Terraform Managed "Reference Architecture Service" Strategy

Collections of services provisioned through the use of Terraform resources, which collectively aim to provide a service with continuity, will be configured such that the code (HCL, YAML, JSON, etc.) describes the desired state of this service, made available for consumption. These service collections when unified to provide constructive capability, will fall into one of two categories both of which must comply with the following principles:
- [Self Contained Repository Deployment Principle](../../principles/self-contained-repository-deployment-principle.md)
- [Single Purpose RepositoryPrinciple](../../principles/self-contained-repository-deployment-principle.md)

#### 1. "Exclusive Service"

This class of service is defined as a service for which there is only one consumer. In this scenario, the git repository that contains the code for this single consumer service shall also contain the means to fully converge the state of this "Exclusive Service".

#### 2. "Shared Service"

This class of service is defined as a service for which there are or will be more than one consumer. In this scenario:

- An implementation of this service shall be described from a single purposed git repository utilizing a poly repository git strategy.
- The specification of how this service is being provided and consumed needs to be explicitly defined. The responsibility of the code describing the desired state ends with the "Shared Service's" uptime and availability for consumption.

### Terraform Provider Strategy

For all possible scenarios, the mainstream stable version of the "Terraform Provider" required to provide resources will be used. When a need arises where this provider does not offer a needed resource or there is a bug with an existing resource, there are two options for adding support for this needed custom resource:

#### Preferred Option (If Infrequent)

1. Create a fork of the latest provider.
2. Follow the project's contribution requirements to add the needed feature or bug fix to the fork.
3. Ensure testing to assert no regression and functionality of the change.
4. Refactor the consuming Terraform module to use this forked provider.
5. Deliver the work to production.
6. Create a Tech Debt Story to get this change merged into the trunk of the provider and released.
7. Upon a successful merge and release in the main public Terraform provider, refactor the consuming code to use this release or greater.

If this change from your fork cannot be merged and there are no other preferable options to resolve the issue, use the provided secondary alternative.

#### Secondary Option (If Frequent or As Mentioned Above)

1. Create a fork of the latest provider.
2. If the provider code in its current state does not allow one to override or extend its function (as it is set private), set the needed areas public in this fork.
3. Ensure this provider is managed by a full pipeline following all general best practices.
4. Ensure on some interval this fork is pulling and merging in updates.
5. Create a custom provider which imports this fork with public methods.
6. Override or extend these methods as needed.
7. Ensure testing to assert no code regression and functionality of the change.
8. Ensure this provider is managed by a full pipeline following all general best practices.
9. Release a new semantic version to be consumed by Terraform modules.
10. Ensure when a new feature or bug fix is added to this custom provider, it is using the latest semantic version of the fork where the methods were made public.

### Terraform Module Strategy

#### Classes of Terraform Modules

##### Primitive Module

**Definition:**
A module that provides the means to describe the state of a single class of a cloud resource such as blob storage or a load balancer. Resource blocks are permitted.

**Requirements:**
- Use of a mature HashiCorp module published in the public Terraform registry must be the first choice.
- Must be client and application agnostic.
- No explicit attributes; all attributes must be set by inputs.
- If custom developed, must be created using the latest version of tf-module-skeleton.
- Must have a pipeline that enforces linting, build test, testing (including pre-functional test using rego and post-functional testing using Terratest).
- Tests must cover all possible features/functions from "no state" to the convergence of the actual state.
- Must be semantically versioned and published using git tags.
- Software Licensing: Apache 2.0

##### Library Module

**Definition:**
An opinionated module that adds additional functionality to a primitive or wrapper module, such as providing an approach to resource naming and tagging. Resource blocks are not permitted.

**Requirements:**
- Must be client and application agnostic.
- No explicit attributes; all attributes must be set by inputs.
- If custom developed, must be created using the tf-module-skeleton.
- Must have a pipeline that enforces linting, build test, testing (including pre-functional test using rego and post-functional testing using Terratest).
- Tests must cover all possible features/functions from "no state" to the convergence of the actual state.
- Must be semantically versioned and published using git tags.
- Software Licensing: Apache 2.0

##### Wrapper Module

**Definition:**
An opinionated module that is only a collection of other primitive modules using the "Everything as a Module Approach." It provides a "Reference Architecture Service" which brings everything needed to provide a single service type to consumers in production.

**Examples:**
- A Kubernetes cluster with all cloud resources, configuration, custom resources definitions, operators, etc., ready for deployments and hosting workloads.
- A CDN, edge lambda, WAF DNS Shield, Blob storage, etc., ready to provide hosting of a static www site with either server-side or client-side rendering.

**Requirements:**
- Must be client and application agnostic.
- Will always be custom developed or use of a prior custom developed wrapper module.
- Must be created using the tf-module-skeleton.
- Must have a pipeline that enforces linting, build test, testing (including pre-functional test using rego and post-functional testing using Terratest).
- Testing of functionality tested in the version of imported modules is generally not needed, but testing the integration of all imported modules is necessary.
- Tests must assert the collection of modules arrived in the expected state provided by the module from "no state" to the convergence of the planned state and the actual state.
- Part of an actual deploy of this module must include testing of its provided service from the perspective of its intended consumer and validation of its health.
- Must be semantically versioned and published using git tags.
- Software Licensing: Apache 2.0

##### Client Specific Wrapper Module

**Definition:**
A module that imports a wrapper module which provides a "Reference Architecture Service." It is created for the purpose of adding in client-specific functionality, such as policies, observability integrations, resource and tagging requirements, and other customizations.

**Requirements:**
- Must be client-specific and application agnostic.
- Will always be custom developed or use of a prior custom developed client wrapper module.
- Must be created using the tf-module-skeleton.
- Must have a pipeline that enforces linting, build test, testing (including pre-functional test using rego and post-functional testing using Terratest).
- Testing of functionality tested in the version of imported modules is generally not needed, but testing the integration of all imported modules is necessary.
- Tests must assert the collection of modules arrived in the expected state provided by the module from "no state" to the convergence of the planned state and the actual state.
- Part of an actual deploy of this module must include testing of its provided service from the perspective of its intended consumer and validation of its health.
- The consumer must provide a health endpoint or some means to verify minimal functionality; the scope here is a "smoke test."
- Must be semantically versioned and published using git tags.
- Software Licensing: as required by the client.

## Implementation of Terraform Module Strategy

### General Objective, Guiding Mission Statement

For each git repository using a poly repository strategy, where its intent is to produce a single artifact providing a single service, the repository must contain everything needed to deliver its service to the end consumer. This includes:

- An "Exclusive Service" provided by resources.
- An end-to-end pipeline.
- All test logic.

An exception is made only in scenarios where the service depends on a "Shared Service" as defined above. In such cases, it will not provide the "Shared Service" but will validate its existence and readiness. It will then contain everything required to deliver itself to the end consumer when consuming this externally provided "Shared Service".

TerraGrunt will exist in every git repository to provide:
- A fully configured pipeline.
- All infrastructure resources needed for the hosting and uptime of the service being provided.
- In some situations, all resources needed to consume a platform for which there is a Terraform Provider.

#### Requirements

- There must be a separate remote state file for each uniquely deployed environment, bootstrapped by TerraGrunt.
- If multiple instance sets of an environment exist where only one can be active, each set must have a separate remote state file.
- Terraform code, aside from HCL required by TerraGrunt, is not permitted.
- Only one wrapper module for the service and one wrapper module for the pipeline are permitted to be referenced by TerraGrunt.
- TerraGrunt HCL must be as DRY as possible.
- Explicit values are not permitted and must be passed in.
- All properties needed by TerraGrunt, or the application must live in "<this git repository name>-properties".

#### Future Improvement Plans

- It is intended that TerraGrunt HCL will be dynamically generated to allow for immutable deployments sets of either "Exclusive or Shared Services".
- In this future state, the meets and bounds of what service is provided and how it is provided must be well-defined.
- There must further be a well-defined strategy of the threshold of where mutation is or is not permitted.
- There must further be a well-defined strategy of where state lives and how this state is carried forward between deployments of immutable instance sets.

### Resource Naming and Tagging Strategy

#### Resource Naming

The specification for resource naming includes:

- In some cases, constraints may not allow this naming standard, especially in Azure.
- The library module tf-module-name can currently output in 8 different formats to accommodate these constraints.
- The scheme must include the capability to add key/value pairs as metadata.
- The key/value pair with the key 'name' is more critical than the resource name, as it will be needed to query for related sets of resources in the future.

Example: 'name: <logical product name>-<region>-<class of env>-<instance of env>', such as 'name: ehdc-us_west_1-prod-000'.

##### Immutable Infrastructure Deployment Instance Sets

- `<logical product name>-<region>-<class of env>-<instance of env>`
- Active and inactive instances, e.g., 'ehdc-us_west_1-prod-000 - active'.

Every infrastructure resource must be named as follows:
- `<logical product name>-<region>-<class of env>-<instance of env>-<cloud resource>-<instance of a resource>`
- Examples: 'ehdc-us_west_1-prod-000-rg-000', 'ehdc-us_west_1-prod-000-s3-000'.

### Terraform Module Git Repository Naming Strategy

#### Example Custom Terraform Provider Repo Name

- tf-provider-nextgen_aws
- tf-provider-nextgen_github

#### Example Primitive Module Repo Name

- tf-module-<primitive resource>
- Examples: tf-module-azstorage, tf-module-gcpstorage

#### Example Wrapper Module Repo Name

- tf-module-<wrapper resource>
- Examples: tf-module-ecs_platform, tf-module-eks_platform

#### Example Library Module Repo Name

- tf-module-<library resource>
- Examples: tf-module-name, tf-module-metadata