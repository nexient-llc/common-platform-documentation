# Helm Library Chart Development Policy

## Overview
This document outlines guidelines for developing Helm library charts within Launch projects.

## Purpose
The purpose of this policy is to ensure consistency, maintainability, and reusability of Helm library charts across Launch projects.

## Guidelines

### Chart Structure
* The chart must include a `Chart.yaml` file that defines the chart metadata.
* The chart must include a `values.yaml` file that contains the default values for the chart.
* The chart must include a `templates` directory that contains the template files for the chart.

### Chart Dependencies
* The library chart must declare its dependencies in the `Chart.yaml` file.
* Dependencies must be specified using the `dependencies` section.
* In general, Library Charts should **not** have dependencies.

### Chart Templates
* No templates should render directly from a Library Chart.
* Templates should be named as follows:
  * `launch`-`<library-chart-name>`-`library`
* Acceptable name examples:
  * `launch-helm-library`
  * `launch-crd-library`
  * `launch-operator-library`

### Documentation
* Provide clear and concise documentation for the Library chart.
* Include a `README.md` file with instructions on how to use the chart, including a reference for all variables that may be passed to the chart. 
  * Chart documentation tools are documented in the [helm-development top-level README](../../README.md#documentation)
* Document any prerequisites or assumptions.

### Testing and Validation
* Write tests to validate the correctness of the chart.
* Use tools like Helm lint and Helm test for automated testing.
* Validate the chart against different Kubernetes versions and environments.
* Chart testing and validation tools are listed in the [helm development top-level README](../../README.md#tools).

## Conclusion
Following these guidelines will help ensure consistency and maintainability of Helm library charts within our project. Please refer to this document when developing new library charts or making changes to existing ones.

For any questions or clarifications, please reach out to the project maintainers.
