# Continuous Deployment

## Introduction

The Kubernetes platform is a flexible and powerful tool for enabling software delivery quality and software execution resiliency. Kubernetes enables a wide-open landscape and it needs good tooling, processes, and policy to realize that potential. 

A key concept that enables software delivery quality is **Continuous Deployment**. 

> It is important to note that we must separate the ideas of *deployment* and *delivery*. *Deployment* is the process of delivering code into an execution environment. *Delivery* is the process of enabling some functionality, capability, or experience that is delivered by software.

A key tool for enabling Continuous Deployment in the Kubernetes space is Helm. By itself, Helm is not intended to be a Continuous Deployment tool. The Helm developers have been quite adamant in support documents that Helm is a Package Manager, not a CD tool. However, it is a key component in delivering a continuous deployment capability.

## Helm Capabilities

For a complete review of Helm, please refer to the [official documentation](https://helm.sh/docs/). 

Launch uses these specific capabilities of Helm for the purposes of enabling Continuous Deployment:

* Composable charts -- The ability to build charts from templated components.
* Reusable charts -- The ability to use composed charts with diverse inputs to generate unique, stable outcomes.
* Deployment management -- The ability to [hook into deployment phases](https://helm.sh/docs/topics/charts_hooks/) for managing deployment quality.
* Deployment non-repudiation -- The ability to render deployment manifests prior to deployment and to keep these manifests as attestations for the deployment.

## Helm as App CD Tool

Continuous Deployment works best in a microservice-based application model. This allows for abstractions to exist that enable flexibility in how components of the overall solution are deployed and engaged in the delivery of the solution.

Launch will maintain these standards when using Helm as the application delivery tool:

* Helm deployment name will be based on microservice name.
  * Additional naming components may be postfixed to allow for immutable deployments with unique deployment names.
* appVersion only variable of consequence from deployment to deployment.
* Immutable deployments will deploy the complete chart into a new namespace for each deployment.
* All environments will use the same umbrella chart.
* `values.yaml` will be stored in properties repo, one per environment.
* `Chart.yaml` will live in app repo
  * define dependencies and versions
  * dependencies will include Launch-managed deployment chart for microservices
  * `appVersion` will be a **magic** value, replaced or inserted at deployment time with artifact version ID
  * `appVersion` propagates into manifests as deployed image tag (and potentially other targets)

### Continuous Deployment Workflow

```mermaid
flowchart TD;
    A[merge] -.-> |app build, test, artifact| B[Tagged artifact deployed
    to registry]
    B --> C[Render full chart from umbrella chart 
    using appVersion from artifact step]
    C --> D[Validate chart (dry-run, kubeconform, etc.)]
    D --> F([Package rendered templates 
    as deployment artifact])
    D --> E[helm upgrade --install ...]
    E --> G[Monitor deployment]
    G --> H{Approve?}
    H --> |Yes| I([Finish deployment])
    H --> |No| J([Rollback])
```

## Helm as Middleware Management Tool

* Use middleware provided helm charts
* Manage our own values.yaml 
* Each M/W component gets its own pipeline and properties repo

TODO: figure out how to converge both use-cases

## Next Steps for Launch use of Helm

* Investigate [Helm Sigstore](https://github.com/sigstore/helm-sigstore?tab=readme-ov-file) plugin for chart signing and provenance 
* Investigate [Helm Secrets](https://github.com/jkroepke/helm-secrets) plugin for managing secrets 
* Investigate [Helm Git](https://github.com/aslafy-z/helm-git) plugin for retrieving values files directly from git.
* Investigate immutable deployments using Helm.

### Open Questions

There are still many open questions about how to best use Helm for Continuous Deployment on Kubernetes. As of this writing, these open questions still need to be explored:

* explore best method for canary or blue/green deployments
  * integration with service mesh
  * some interactivity required to approve move from partial to full deployment, or rejection of deployment.
* explore triggering event based on new tag deployed to container registry
* explore upgrading chart version
  * implications to app availability, disruptions, etc
* explore upgrading chart version, but not upgrading app version
  * probably will make policy against this, but need to understand it
* explore secret management
  * how to originate secrets into key vault / secret manager
    * most likely via IaC, not via Helm
  * internal (not shared) vs external (shared) secrets
* explore how to know if data schema is appropriate for app version to be deployed
* explore how to deploy "data"
  * helm chart as "job"
  * includes helm metadata with deployed version?
  * integrated with app deployment chart, or separate?
    * initial thought is to have them separate
  * must have data schema strategy that allows for N-1, N, N+1 -- [Evolutionary Database](https://www.martinfowler.com/articles/evodb.html)
