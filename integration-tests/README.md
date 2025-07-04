# Integration Tests

* [Integration Tests](#integration-tests)
  * [Introduction](#introduction)
  * [Prerequisites](#prerequisites)
  * [Test cases](#test-cases)
    * [Shared file](#shared-file)
  * [Deployment](#deployment)
    * [Configuration](#configuration)
      * [Jaeger Integration Tests Parameters](#jaeger-integration-tests-parameters)
    * [Manual Deployment](#manual-deployment)
      * [Installation](#installation)
      * [Uninstalling](#uninstalling)

## Introduction

This guide covers the necessary steps to install and execute Jaeger service tests on Kubernetes/Openshift using Helm.
The chart installs Jaeger Integration Tests service and pod in Kubernetes/Openshift.

## Prerequisites

* Kubernetes 1.25+ or OpenShift 3.11+
* `kubectl` 1.25+
* Helm 3.0+

## Test cases

1. [Smoke tests](robot/tests/smoke/smoke.robot)

   * Check Deployments
     This is test check that indicated namespace hasn't inactive deployments for Collector and Query deployments.

   * Check Collector Pods Are Running
     This is test check that all pods from Collector deployment has been running state.

   * Check Query Pods Are Running
     This is test check that all pods from Query deployment are in the running state.

   * Jaeger can serve spans
     This test check health status from Jaeger and send POST request with generated spans (template of the span can be found
     [spans.json](robot/tests/libs/resources/spans.json)).
     After that will be checked that span was added to Jaeger (Will be sent GET request to Jaeger).

2. [Spans generator](robot/tests/spans_generator/generate.robot)

   * Send spans
     This test provides sending a lot of same spans (with different timestamp only) to Jaeger.
     In deployment parameters you need to indicate host for get spans, count for sending and time between sending.

3. [HA tests](robot/tests/tests_ha/ha.robot)

   * Reboot query pod
     This test check the integrity of the spans if there is a loss of Query Pod.

   * Reboot collector pods
     This test check the availability of sending spans to Jaeger if there is a failure of some Collector pods.
     The Collector pods will be reboot one by one.

4. [Hardcoded Images](robot/tests/image_tests/image_tests.robot)

   * Test Hardcoded Images
     This test compare images in pods with images from MF. Included in the `smoke` tag

### Shared file

The `shared.robot` [file](robot/tests/shared/shared.robot)
contains main keywords and main settings. For example, settings for retries and time between retries,
connection parameters, convectors, etc.

## Deployment

Jaeger integration tests installation is based on Helm Chart directory.

### Configuration

This section provides the list of parameters required for Jaeger Integration Tests installation and execution.

#### Jaeger Integration Tests Parameters

<!-- markdownlint-disable line-length -->
| Parameter                                             | Type    | Mandatory | Default value                                              | Description                                                                                                                                                                                                                                                                                                             |
| ----------------------------------------------------- | ------- | --------- | ---------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `integrationTests.service.name`                       | string  | no        | jaeger-integration-tests-runner                            | The name of Jaeger Integration Tests service                                                                                                                                                                                                                                                                            |
| `integrationTests.serviceAccount.create`              | boolean | no        | true                                                       | Specifies whether service account for Jaeger Integration Tests is to be deployed or not                                                                                                                                                                                                                                 |
| `integrationTests.serviceAccount.name`                | string  | no        | jaeger-integration-tests                                   | The name of the service account that is used to deploy Jaeger Integration Tests. If this parameter is empty, the service account, the required role, role binding are created automatically with default names (`jaeger-integration-tests`)                                                                             |
| `integrationTests.install`                            | boolean | no        | false                                                      | Specifies whether Jaeger Integration Tests Service should be installed or not                                                                                                                                                                                                                                           |
| `integrationTests.image`                              | string  | no        | -                                                          | The Docker image of Jaeger Integration Tests Service                                                                                                                                                                                                                                                                    |
| `integrationTests.tags`                               | string  | no        | smoke                                                      | Tags combined together with `AND`, `OR` and `NOT` operators that select test cases to run. You can use the "smoke", "generator" and "ha" tags to run the appropriate tests. Or a combination of both, for example `smokeORha` to run both smoke and ha tests                                                            |
| `integrationTests.linkForGenerator`                   | string  | no        | `http://jaeger-collector:9411`                             | Link to host which can get spans in Zipkin format                                                                                                                                                                                                                                                                       |
| `integrationTests.generateCount`                      | integer | no        | 10                                                         | The number of spans which will be sent, 10 by default                                                                                                                                                                                                                                                                   |
| `integrationTests.waitingTime`                        | string  | no        | 500ms                                                      | The waiting time between sending, by default 500ms. Time format can be found in [official robot documentation](https://robotframework.org/robotframework/latest/libraries/BuiltIn.html#Sleep)                                                                                                                           |
| `integrationTests.resources.requests.memory`          | string  | no        | 256Mi                                                      | The minimum amount of memory the container should use. The value can be specified with SI suffixes (E, P, T, G, M, K, m) or their power-of-two-equivalents (Ei, Pi, Ti, Gi, Mi, Ki)                                                                                                                                     |
| `integrationTests.resources.requests.cpu`             | string  | no        | 50m                                                        | The minimum number of CPUs the container should use                                                                                                                                                                                                                                                                     |
| `integrationTests.resources.limits.memory`            | string  | no        | 256Mi                                                      | The maximum amount of memory the container can use. The value can be specified with SI suffixes (E, P, T, G, M, K, m) or their power-of-two-equivalents (Ei, Pi, Ti, Gi, Mi, Ki)                                                                                                                                        |
| `integrationTests.resources.limits.cpu`               | string  | no        | 400m                                                       | The maximum number of CPUs the container can use                                                                                                                                                                                                                                                                        |
| `integrationTests.affinity`                           | object  | no        | -                                                          | The affinity scheduling rules. The value should be specified in json format. The parameter can be empty                                                                                                                                                                                                                 |
| `integrationTests.statusWriting.enabled`              | boolean | no        | false                                                      | Specifies whether to write status to custom resource                                                                                                                                                                                                                                                                    |
| `integrationTests.statusWriting.isShortStatusMessage` | boolean | no        | true                                                       | Specifies the size of integration test status message                                                                                                                                                                                                                                                                   |
| `integrationTests.statusWriting.onlyIntegrationTests` | boolean | no        | true                                                       | Specifies to deploy only integration tests without any component (component was installed before)                                                                                                                                                                                                                       |
| `integrationTests.statusWriting.customResourcePath`   | string  | no        | apps/v1/jaeger/deployments/jaeger-integration-tests-runner | Path to Custom Resource that should be used to write status of integration-tests execution. The value is a field from k8s entity selfLink without `apis` prefix and `namespace` part. The path should be composed according to the following template: `<group>/<apiversion>/<namespace>/<plural>/<customResourceName>` |
<!-- markdownlint-enable line-length -->

### Manual Deployment

#### Installation

To deploy Jaeger integration tests with Helm you need to customize the `values.yaml` file. For example:

```yaml
integrationTests:
  install: true
  image: "ghcr.io/netcracker/jaeger-integration-tests:main"
  tags: "smokeORha"
  linkForGenerator: "https://jaeger-collector-host"
  generateCount: 10
  waitingTime: 500ms
  resources:
    requests:
      memory: 256Mi
      cpu: 50m
    limits:
      memory: 256Mi
      cpu: 400m
  service:
    name: jaeger-integration-tests-runner
  serviceAccount:
    create: true
    name: "jaeger-integration-tests"
```

To deploy the service you need to execute the following command:

```bash
helm install -n ${NAMESPACE} ${RELEASE_NAME} ./jaeger-integration-tests 
```

where:

* `${RELEASE_NAME}` is the Helm Chart release name and the name of the Jaeger integration tests.
For example, `jaeger-integration-tests`.
* `${NAMESPACE}` is the Kubernetes namespace or Openshift workspace to deploy Jaeger service integration tests.
For example, `jaeger`.

You can monitor the deployment process in the Kubernetes/Openshift dashboard or using `kubectl` in the command line:

```bash
kubectl get pods
```

#### Uninstalling

To uninstall Jaeger integration tests from Kubernetes/Openshift you need to execute the following command:

```bash
helm delete ${RELEASE_NAME} -n ${NAMESPACE}
```

where:

* `${RELEASE_NAME}` is the Helm Chart release name and the name of the Jaeger integration tests.
For example, `jaeger-integration-tests`.
* `${NAMESPACE}` is the Kubernetes namespace or Openshift workspace to deploy Jaeger service integration tests.
For example, `jaeger`.

The command uninstalls all the Kubernetes resources associated with the chart and deletes the release.
