

# setup-rh-build-of-keycloak

[![License](https://img.shields.io/badge/License-Apache_2.0-blue.svg)](https://opensource.org/licenses/Apache-2.0)
[![Release Charts](https://github.com/websecwrc/helm-charts/actions/workflows/release.yml/badge.svg)](https://github.com/websecwrc/helm-charts/actions/workflows/release.yml)

  ![Version: 1.0.0](https://img.shields.io/badge/Version-1.0.0-informational?style=flat-square)

 

  ## Description

  Deploy and configure the operator Red Hat Build of Keycloak

This chart will perform:
1. Installation of RH build of Keycloak Operator
2. Configuration of Keycloak instance
3. Installation of an example database (optional)
4. Reqesting a Certificate using cert-manager (optional)

## Dependencies

This chart has the following dependencies:

| Repository | Name | Version |
|------------|------|---------|
| https://charts.stderr.at/ | cert-manager | ~1.0.0 |
| https://charts.stderr.at/ | helper-operator | ~1.0.21 |
| https://charts.stderr.at/ | helper-status-checker | ~4.0.0 |
| https://charts.stderr.at/ | rh-build-keycloak | ~1.0.0 |
| https://charts.stderr.at/ | tpl | ~1.0.0 |

## Maintainers

| Name | Email | Url |
| ---- | ------ | --- |
| tjungbauer | <tjungbau@redhat.com> | <https://blog.stderr.at/> |

## Sources
Source:
* <https://github.com/websecwrc/helm-charts>
* <https://charts.stderr.at/>
* <https://github.com/websecwrc/openshift-clusterconfig-gitops>

Source code: https://github.com/websecwrc/openshift-clusterconfig-gitops/tree/main/clusters/management-cluster/setup-rh-build-of-keycloak

## Example values files

```yaml
---
namespace: &namespace "keycloak"
ssohostname: &ssohostname sss.apps-crc.testing

cert-manager:
  enabled: true

  certificates:
    enabled: true

    # List of certificates
    certificate:
      - name: keycloak-certificate
        enabled: true
        namespace: *namespace
        syncwave: "0"
        secretName: keycloak-certificate

        dnsNames:
          - *ssohostname

        # Reference to the issuer that shall be used.
        issuerRef:
          name: letsencrypt-prod
          kind: ClusterIssuer

# -- The following section contains the configuration for the Red Hat build of Keycloak.
rh-build-keycloak:
  enabled: true

  namespace:
    name: *namespace
    create: true

  keycloak:
    name: example-keycloak
    namespace: *namespace

    hostname:
      hostname: *ssohostname

    http:
      tlsSecret: "keycloak-certificate"

    db:
      use_example_db_sta: true
      exmple_db_user: testuser
      example_db_pass: ""

# Install Operator Compliance Operator
# Deploys Operator --> Subscription and Operatorgroup
# Syncwave: 0
helper-operator:
  operators:
    rhbk-operator:
      enabled: true
      syncwave: '0'
      namespace:
        name: *namespace
        create: false
      subscription:
        channel: stable-v22
        approval: Automatic
        operatorName: rhbk-operator
        source: redhat-operators
        sourceNamespace: openshift-marketplace
      operatorgroup:
        create: true
        notownnamespace: false

helper-status-checker:
  enabled: true

  checks:
    - operatorName: rhbk-operator
      namespace:
        name: rhbk-operator
      serviceAccount:
        name: "status-checker-rhbk"
```

----------------------------------------------
Autogenerated from chart metadata using [helm-docs v1.12.0](https://github.com/norwoodj/helm-docs/releases/v1.12.0)
