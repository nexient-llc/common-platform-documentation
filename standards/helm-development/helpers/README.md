# Helm Chart Helpers Development Policy

## Introduction

Helm Chart helpers are small template functions, usually used to manipulate an existing string or value to meet a different requirements. Generally, helpers are used to convert one data "structure" to another. Examples of useful helpers include converting a key, value map into an environment variable manifest structure, removing unacceptable characters from a string, or generating a small pattern of labels that will be reused many times within a chart.

Helpers are named templates. Helpers are differentiated from named templates by the fact that, in general, helpers are not intended to render actionable kubernetes manifests.

Helpers can be included in Library or Deployment charts.

## Naming

Helpers are structurally the same as named templates. Helpers should be named in a way that quickly conveys their purpose without being overly verbose. 

If a helper is defined in a Library Chart, the name of the helper should be prepended with the name of the library chart.

## Documentation

Each helpers should have a comment block that contains at least this information:

* Purpose of the helper
* Expected inputs
* Expected outputs

## Testing

Helpers should be tested along side all other templates included in the chart. 

---
**Document Revision History**
| Date | Version | Author |Notes |
| --- | --- | --- | --- |
| 2024-03-15 | 1.0 | Ben Vaughan | Initial Release |
