

# setup-openshift-logging

[![License](https://img.shields.io/badge/License-Apache_2.0-blue.svg)](https://opensource.org/licenses/Apache-2.0)
[![Linting](https://github.com/websecwrc/openshift-clusterconfig-gitops/actions/workflows/linting.yml/badge.svg)](https://github.com/websecwrc/openshift-clusterconfig-gitops/actions/workflows/linting.yml)
[![Release Charts](https://github.com/websecwrc/helm-charts/actions/workflows/release.yml/badge.svg)](https://github.com/websecwrc/helm-charts/actions/workflows/release.yml)

  ![Version: 1.0.1](https://img.shields.io/badge/Version-1.0.1-informational?style=flat-square)

 

  ## Description

  Installs and configures OpenShift Logging by deploying Logging and Loki Operator and configuring them accordingly. Example configuration is creating a Bucket using OpenShift Data Foundation.

This "wrapper" Helm Chart is used to deploy and configure OpenShift Logging with the LokiStack using a GitOps approach.

The values.yaml provides an example of possible settings.

**NOTE**: Verify the article [Installing OpenShift Logging Using GitOps](https://blog.stderr.at/gitopscollection/2024-05-24-install-openshift-logging/) for more detailed information.

There are 6(!) additional Charts that are required as a dependency when you want to use ODF and fully automate the deployment.
Verify the README and/or the values files for further information.

## Dependencies

This chart has the following dependencies:

| Repository | Name | Version |
|------------|------|---------|
| https://charts.stderr.at/ | helper-loki-bucket-secret | ~1.0.0 |
| https://charts.stderr.at/ | helper-lokistack | ~1.0.0 |
| https://charts.stderr.at/ | helper-objectstore | ~1.0.0 |
| https://charts.stderr.at/ | helper-operator | ~1.0.18 |
| https://charts.stderr.at/ | helper-status-checker | ~4.0.0 |
| https://charts.stderr.at/ | openshift-logging | ~2.0.0 |

## Maintainers

| Name | Email | Url |
| ---- | ------ | --- |
| tjungbauer | <tjungbau@redhat.com> | <https://blog.stderr.at/> |

## Sources
Source:
* <https://github.com/websecwrc/helm-charts>
* <https://charts.stderr.at/>
* <https://github.com/websecwrc/openshift-clusterconfig-gitops>

Source code: https://github.com/websecwrc/openshift-clusterconfig-gitops/tree/main/clusters/management-cluster/setup-openshift-logging

## Example values files

```yaml
---
logging-namespace: &log-namespace openshift-logging
bucketname: &bucketname logging-bucket
lokisecret: &loki-secret-name logging-loki-s3
storageclassname: &storageclassname logging-bucket-storage-class
lokistack: &lokistackname logging-loki
loki: &channel-loki stable-5.8
logging: &channel-logging stable

######################################
# SUBCHART: helper-operator
# Operators that shall be installed.
######################################
helper-operator:

  # -- Configure console plugins for OpenShift.
  # @default -- ""
  console_plugins:
    # -- Enable console plugin configuration.
    # @default -- false
    enabled: true
    # -- List of console plugins to configure. Each list item will be added to the OpenShift UI.
    # @default -- empty
    plugins:
      - logging-view-plugin

  operators:
    cluster-logging-operator:
      # -- Enabled yes/no
      # @default -- false
      enabled: true
      # -- Syncwave for the operator deployment
      # @default -- 0
      syncwave: '0'

      namespace:
        # -- The Namespace the Operator should be installed in.
        name: *log-namespace
        # -- Create the Namesoace yes/no.
        # @default -- false
        create: true
      # -- Definition of the Operator Subscription
      # @default -- ""
      subscription:
        # -- Channel of the Subscription
        # @default -- stable
        channel: *channel-logging
        # -- Source of the Operator
        # @default -- redhat-operators
        source: redhat-operators
        # -- Update behavior of the Operator. Manual/Automatic
        # @default -- Automatic
        approval: Automatic
        # -- Name of the Operator
        # @default -- "empty"
        operatorName: cluster-logging
        # -- Namespace of the source
        # @default -- openshift-marketplace
        sourceNamespace: openshift-marketplace
      operatorgroup:
        # -- Create an Operatorgroup object
        # @default -- false
        create: true
        # -- Monitor own Namespace. For some Operators no `targetNamespaces` must be defined
        # @default -- false
        notownnamespace: false

    loki-operator:
      enabled: true
      namespace:
        name: openshift-operators-redhat
        create: true
      subscription:
        channel: *channel-loki
        approval: Automatic
        operatorName: loki-operator
        source: redhat-operators
        sourceNamespace: openshift-marketplace
      operatorgroup:
        create: true
        notownnamespace: true

########################################
# SUBCHART: helper-status-checker
# Verify the status of a given operator.
########################################
helper-status-checker:

  # -- Enable or disable LokiStack configuration
  # @default -- false
  enabled: true

  # -- Enable automatic approval of an InstallPlan. Useful if the installation must be approved manually and you want to initially deploy the Operator using GitOps.
  # @default -- false
  approver: false

  # List of checks that shall be performed.
  checks:
       # -- Name of operator to check. Use the value of the currentCSV (packagemanifest) but WITHOUT the version !!
    - operatorName: cluster-logging
      subscriptionName: cluster-logging-operator

      # -- If the Operator is not yet ready wait this amount of seconds.
      # @default -- 20
      sleeptimer: 20

      # -- Maximum number of retries before the checks will fail
      # @default -- 20
      maxretries: 20

      # -- Namespace where the status-checker Job shall be scheduled.
      namespace:
        name: openshift-logging

      # -- Syncwave for the status-check Job
      # @default -- 0
      syncwave: 1

      serviceAccount:
        # -- Name of the Service Account.
        name: "status-checker-logging"

    - operatorName: loki-operator
      namespace:
        name: openshift-operators-redhat
      syncwave: 1

      serviceAccount:
        name: "status-checker-loki"

########################################
# SUBCHART: helper-objectstore

# A helper chart that simply creates another backingstore for logging.
# This is a chart in a very early state, and not everything can be customized for now.
# It will create the objects:
#  - BackingStore
#  - BackingClass
#  - StorageClass

# NOTE: Currently only PV type is supported
########################################
helper-objectstore:

  # -- Enable objectstore configuration
  # @default -- false
  enabled: true

  # -- Syncwave for Argo CD
  # @default - 1
  syncwave: 1

  # -- Name of the BackingStore
  backingstore_name: logging-backingstore

  # -- Size of the BackingStore that each volume shall have.
  backingstore_size: 700Gi

  # -- CPU Limit for the Noobaa Pod
  # @default -- 500m
  limits_cpu: 500m

  # -- Memory Limit for the Noobaa Pod.
  # @default -- 2Gi
  limits_memory: 2Gi

  pvPool:
    # -- Number of volumes that shall be used
    # @default -- 1
    numOfVolumes: 1

    # Type of BackingStore. Currently pv-pool is the only one supported by this Helm Chart.
    # @default -- pv-pool
    type: pv-pool

  # -- The StorageClass the BackingStore is based on
  baseStorageClass: gp3-csi

  # -- Name of the StorageClass that shall be created for the bucket.
  storageclass_name: *storageclassname

  # Bucket that shall be created
  bucket:
    # -- Shall a new bucket be enabled?
    # @default -- false
    enabled: true

    # -- Name of the bucket that shall be created
    name: *bucketname

    # -- Target Namespace for that bucket.
    namespace: *log-namespace

    # -- Syncwave for bucketclaim creation. This should be done very early, but it depends on ODF.
    # @default -- 2
    syncwave: 2

    # -- Name of the storageclass for our bucket
    # @default -- openshift-storage.noobaa.io
    storageclass: *storageclassname

##############################################
# SUBCHART: helper-loki-bucket-secret
# Creates a Secret that Loki requires
#
# A Kubernetes Job is created, that reads the
# data from the Secret and ConfigMap and
# creates a new secret for loki.
##############################################
helper-loki-bucket-secret:
  # -- Enable Job to create a Secret for LokiStack.
  # @default -- false
  enabled: true

  # -- Syncwave for Argo CD.
  # @default -- 3
  syncwave: 3

  # -- Namespace where LokiStack is deployed and where the Secret shall be created.
  namespace: *log-namespace

  # -- Name of Secret that shall be created.
  secretname: *loki-secret-name

  # Bucket Configuration
  bucket:
    # -- Name of the Bucket shall has been created.
    name: *bucketname

########################################
# SUBCHART: helper-lokistack
# Renders a LokiStack object.
########################################
# Subchart with values to render the LokiStack object
helper-lokistack:
  # -- Enable or disable LokiStack configuration
  # @default -- false
  enabled: true

  # -- Name of the LokiStack object
  name: *lokistackname

  # -- Namespace of the LokiStack object
  namespace: *log-namespace

  # -- Syncwave for the LokiStack object.
  # @default -- 3
  syncwave: 3

  # -- This is for log streams only, not the retention of the object store. Data retention must be configured on the bucket.
  global_retention_days: 4

  # storage settings
  storage:

    # -- Size defines one of the supported Loki deployment scale out sizes.
    # Can be either:
    #   - 1x.demo
    #   - 1x.extra-small (Default)
    #   - 1x.small
    #   - 1x.medium
    # @default -- 1x.extra-small
    # size: 1x.extra-small

    # Secret for object storage authentication. Name of a secret in the same namespace as the LokiStack custom resource.
    secret:

      # -- Name of a secret in the namespace configured for object storage secrets.
      name: *loki-secret-name

      # Type of object storage that should be used
      # Can bei either:
      #  - swift
      #  - azure
      #  - s3 (default)
      #  - alibabacloud
      #  - gcs
      # -- Type of object storage that should be used
      # @default -- s3
      # type: s3

    # Schemas for reading and writing logs.
    # schemas:
      # -- Version for writing and reading logs.
      # Can be v11 or v12
      # @default -- v12
      #  - version: v12

      # -- EffectiveDate is the date in UTC that the schema will be applied on. To ensure readibility of logs, this date should be before the current date in UTC.
      # @default -- 2022-06-01
      #    effectiveDate: "2022-06-01"

  # -- Storage class name defines the storage class for ingester/querier PVCs.
  # @default -- gp3-csi
  storageclassname: gp3-csi

  # -- Mode defines the mode in which lokistack-gateway component will be configured.
  # Can be either: static (default), dynamic, openshift-logging, openshift-network
  # @default -- static
  mode: openshift-logging

  # -- Control pod placement for LokiStack components. You can define a list of tolerations for the following components:
  # compactor, distributer, gateway, indexGateway, ingester, querier, queryFrontend, ruler
  podPlacements:
    # Pod placement of compactor
    compactor:
      tolerations:
        - effect: NoSchedule
          key: node-role.kubernetes.io/infra
          operator: Equal
          value: reserved
        - effect: NoExecute
          key: node-role.kubernetes.io/infra
          operator: Equal
          value: reserved
    # Pod placement of distributor
    distributor:
      tolerations:
        - effect: NoSchedule
          key: node-role.kubernetes.io/infra
          operator: Equal
          value: reserved
        - effect: NoExecute
          key: node-role.kubernetes.io/infra
          operator: Equal
          value: reserved
    # Pod placement of gateway
    gateway:
      tolerations:
        - effect: NoSchedule
          key: node-role.kubernetes.io/infra
          operator: Equal
          value: reserved
        - effect: NoExecute
          key: node-role.kubernetes.io/infra
          operator: Equal
          value: reserved
    # Pod placement of indexGateway
    indexGateway:
      tolerations:
        - effect: NoSchedule
          key: node-role.kubernetes.io/infra
          operator: Equal
          value: reserved
        - effect: NoExecute
          key: node-role.kubernetes.io/infra
          operator: Equal
          value: reserved
    # Pod placement of ingester
    ingester:
      tolerations:
        - effect: NoSchedule
          key: node-role.kubernetes.io/infra
          operator: Equal
          value: reserved
        - effect: NoExecute
          key: node-role.kubernetes.io/infra
          operator: Equal
          value: reserved
    # Pod placement of querier
    querier:
      tolerations:
        - effect: NoSchedule
          key: node-role.kubernetes.io/infra
          operator: Equal
          value: reserved
        - effect: NoExecute
          key: node-role.kubernetes.io/infra
          operator: Equal
          value: reserved
    # Pod placement of queryFrontend
    queryFrontend:
      tolerations:
        - effect: NoSchedule
          key: node-role.kubernetes.io/infra
          operator: Equal
          value: reserved
        - effect: NoExecute
          key: node-role.kubernetes.io/infra
          operator: Equal
          value: reserved
    # Pod placement of ruler
    ruler:
      tolerations:
        - effect: NoSchedule
          key: node-role.kubernetes.io/infra
          operator: Equal
          value: reserved
        - effect: NoExecute
          key: node-role.kubernetes.io/infra
          operator: Equal
          value: reserved

########################################
# SUBCHART: openshift-logging
# Finally, configure openshift-logging.
########################################
openshift-logging:

  loggingConfig:
    enabled: true
    syncwave: '4'

    # Indicator if the resource is 'Managed' or 'Unmanaged' by the operator
    # managementState: Managed

    # Specification of the Log Storage component for the cluster
    logStore:
      # The Type of Log Storage to configure. The operator currently supports either using ElasticSearch managed by elasticsearch-operator or Loki managed by loki-operator (LokiStack) as a default log store.
      type: lokistack

      # Name of the LokiStack resource.
      lokistack: *lokistackname

      retentionPolicy:
        application:
          maxAge: 4d
        audit:
          maxAge: 4d
        infra:
          maxAge: 4d

      visualization:
        # The type of Visualization to configure
        # Could be either Kibana or ocp-console
        type: ocp-console

      collection:
        # The type of Log Collection to configure
        # Vector in case of Loki...
        type: vector

        # The resource requirements for the collector
        resources: {}
        #   limits:
        #     cpu: '500m'
        #     memory: '1Gi'
        #     ephemeral-storage: '50Mi'
        #   requests:
        #     cpu: '500m'
        #     memory: '1Gi'
        #     ephemeral-storage: '500Mi'

        # Define the tolerations the Pods will accept
        # tolerations:
        #  - effect: NoSchedule
        #    key: infra
        #    operator: Equal
        #    value: 'reserved'
```

----------------------------------------------
Autogenerated from chart metadata using [helm-docs v1.12.0](https://github.com/norwoodj/helm-docs/releases/v1.12.0)
