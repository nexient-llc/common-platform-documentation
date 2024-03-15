# Helm Deployment Chart Development Standards

## Introduction

This document outlines the development standards for Helm deployment charts. These standards are designed to ensure consistency, maintainability, and best practices in chart development. In general, Launch standards are a superset of [Helm Chart Best Practices](https://helm.sh/docs/chart_best_practices/). Any variations from the Helm Chart Best Practices will be recorded in this document.

## Table of Contents

- [Helm Deployment Chart Development Standards](#helm-deployment-chart-development-standards)
  - [Introduction](#introduction)
  - [Table of Contents](#table-of-contents)
  - [Chart Naming](#chart-naming)
  - [Chart Versioning](#chart-versioning)
  - [Application Versioning](#application-versioning)
  - [Chart Structure](#chart-structure)
  - [Values file](#values-file)
  - [Variable Naming](#variable-naming)
  - [Templates and Helpers](#templates-and-helpers)
  - [Dependencies](#dependencies)
  - [Testing and Validation](#testing-and-validation)
  - [Documentation](#documentation)


## Chart Naming

* The chart name is defined in the `Chart.yaml` file. 
* The chart name must be concise yet descriptive of the purpose of the chart.
* The chart name should include `chart_` and the chart type as a prefix.
* The chart suffix should be a generic name for the workload type that the chart deploys.
* Example of good chart names are these:
  * `chart_deployment-containerized_app`
  * `chart_job-database_migration`
  * `chart_library-launch`
* Chart names should be unique.

## Chart Versioning

* The chart version is defined in the `Chart.yaml` file. 
* Any value that you commit in the `version` field of the `Chart.yaml` file will be overwritten by the build and deployment automation.
* Chart versions must conform to [Semver 2.0](https://semver.org/spec/v2.0.0.html)
* The "source of truth" for a chart version is the git tag from which a chart is built and deployed.
* Automation will create the git tags which will then drive the chart versions and release versions that are published to the chart repository.

## Application Versioning

* The `appVersion` field in `Chart.yaml` will be replaced at deployment time with the version of the application to be deployed in the pipeline.
* All references to the application version within the chart (image tags, etc.) should refer to the `appVersion`.

## Chart Structure

* The chart directory structure should conform to the [Helm Chart File Structure Standard](https://helm.sh/docs/topics/charts/#the-chart-file-structure)
* The `Chart.yaml` file should contain essential metadata about the chart. See the [official documentation](https://helm.sh/docs/topics/charts/#the-chartyaml-file) for the complete chart reference.
* A `README.md` should be present that contains essential documentation about the chart. See [Documentation](#documentation) for more about the content of the README.

## Values file

* Use the `values.yaml` file to define configurable values for the chart.
* Document the purpose and usage of each configurable value using [helm-docs](https://github.com/norwoodj/helm-docs) documentation tool.
* A valid values file should be published with the chart that is capable of deploying a generic service.

## Variable Naming

* Values variable names must conform to [Helm Values Conventions](https://helm.sh/docs/chart_best_practices/values/). 

## Templates and Helpers

* Any Templates or Helpers defined in a deployment chart must be named with a prefix of the chart name. 

## Dependencies

* Declare and manage chart dependencies using the `dependencies` structure in the `Chart.yaml` file.
* Document the purpose and usage of each dependency.
* Use semantic versioning for chart dependencies.
* The same chart may be depended upon multiple times by defining an `alias` in each definition.
* The same version of a chart **must** be used if a chart is depended upon multiple times.

## Testing and Validation

* Write tests to validate the correctness of the chart.
* Use tools like Helm lint and Helm test for automated testing.
* Validate the chart against different Kubernetes versions and environments.
* Chart testing and validation tools are listed in the [helm development top-level README](../../README.md#tools).

## Documentation

* Provide clear and concise documentation for the chart.
* Include a `README.md` file with instructions on how to use and configure the chart, including a reference for all variables that may be passed to the chart. 
  * Chart documentation tools are documented in the [helm-development top-level README](../../README.md#documentation)
* Document any prerequisites or assumptions.
