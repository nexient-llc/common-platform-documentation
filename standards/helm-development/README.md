# Helm Chart Tools and Best Practices

## Introduction

This document contains Best Practices and tools that the Launch Platform Team uses to develop and maintain Helm charts. In general, Launch standards are a superset of [Helm Chart Best Practices](https://helm.sh/docs/chart_best_practices/). Any variations from the Helm Chart Best Practices will be recorded in this document.

## Helm Development Standards Documents

* [Helm Deployment Charts](charts/deployment/README.md)
* [Helm Library Charts](charts/library/README.md)
* [Helm Template Helpers](helpers/README.md)

## Tools

### Linting

The build in `helm lint` command is useful for basic chart linting.

[yamllint](https://github.com/adrienverge/yamllint) is used for linting `values.yaml` and other yaml files.

### Validation

[kubeconform](https://github.com/yannh/kubeconform) is used for validating rendered templates against specified versions of the Kubernetes API.

### Documentation

[helm-docs](https://github.com/norwoodj/helm-docs) is used for auto-generating documentation from doc strings in the `values.yaml` file.

## Best Practices

### Umbrella Charts

An umbrella chart is a "chart of charts" -- that is, it defines a collection of charts and provides inputs to those charts to accomplish the desired outcome. Umbrella charts are meant to be the only chart that is maintained with an application. The chart should be simply a `Chart.yaml` and an associated `values.yaml`. The `Chart.yaml` would define dependencies that include deployment charts.

> NOTE: An Umbrella Chart should never include a library chart as a dependency.

It is considered a [Best Practice](https://helm.sh/docs/howto/charts_tips_and_tricks/#complex-charts-with-many-dependencies) to create complex deployments using umbrella charts to coordinate many sub-charts.

### Deployment Charts

Conforming with [Helm Best Practices](https://helm.sh/docs/chart_template_guide/subcharts_and_globals/), deployment charts should be able to be deployed on their own, without dependency requirements from sibling charts. It is not possible to pass an output from one child chart to the input of the parent or another child chart.

---
**Document Revision History**
| Date | Version | Author |Notes |
| --- | --- | --- | --- |
| 2024-03-15 | 1.0 | Ben Vaughan | Initial Release |
