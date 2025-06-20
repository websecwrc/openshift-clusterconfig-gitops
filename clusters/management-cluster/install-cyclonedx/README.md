

# install-cyclonedx

[![License](https://img.shields.io/badge/License-Apache_2.0-blue.svg)](https://opensource.org/licenses/Apache-2.0)
[![Linting](https://github.com/websecwrc/openshift-clusterconfig-gitops/actions/workflows/linting.yml/badge.svg)](https://github.com/websecwrc/openshift-clusterconfig-gitops/actions/workflows/linting.yml)
[![Release Charts](https://github.com/websecwrc/helm-charts/actions/workflows/release.yml/badge.svg)](https://github.com/websecwrc/helm-charts/actions/workflows/release.yml)

  ![Version: 1.0.0](https://img.shields.io/badge/Version-1.0.0-informational?style=flat-square)

 

  ## Description

  Install Cyclonedx to store SBOMs

CycloneDX provides advanced, supply chain capabilities for cyber risk reduction. We are using the Software Bill of Material (SBOM) parts.
SBOM is a complete and accurate inventory of all first-party and third-party components is essential for risk identification.

This chart will install CycloneDX BOM Repo server, which enables you to store SBOM inventories on your cluster.
It uses the Chart [cyclonedx](https://github.com/websecwrc/helm-charts/tree/main/charts/cyclonedx).

For detailed information check: [CycloneDX SBOM](https://cyclonedx.org/capabilities/sbom/)

For an example of how to use it during a pipeline run check: [Generating an SBOM](https://blog.stderr.at/securesupplychain/2023-06-22-securesupplychain-step7/)

## Dependencies

This chart has the following dependencies:

| Repository | Name | Version |
|------------|------|---------|
| https://charts.stderr.at/ | cyclonedx | ~1.0.0 |

## Maintainers

| Name | Email | Url |
| ---- | ------ | --- |
| tjungbauer | <tjungbau@redhat.com> | <https://blog.stderr.at/> |

## Sources
Source:
* <https://github.com/websecwrc/helm-charts>
* <https://charts.stderr.at/>
* <https://github.com/websecwrc/openshift-clusterconfig-gitops>

Source code:

## Parameters

## Values

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| namespace.create | bool | false | Create Namespace yes or not |
| namespace.name | string | `"cyclonedx"` | Name of the Namespace |

## Example values

```yaml
---
namespace:
  create: true
  name: cyclonedx
```

----------------------------------------------
Autogenerated from chart metadata using [helm-docs v1.12.0](https://github.com/norwoodj/helm-docs/releases/v1.12.0)
