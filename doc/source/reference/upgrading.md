# Upgrading Seldon Core

This page provides with instructions on how to upgrade from previous versions. Each of these have to be run sequentially if it's expected for previous running models to be upgraded without disruption (i.e if you are running version 0.4.2, you will first have to upgrade to 0.5.2 and then to 1.1, etc).

If you were running our Openshift 0.4.2 certified operator and are looking to upgrade to our 1.1 certified operator, you will also need to follow the "upgrading process" steps in the "Upgrading to 0.5.2 from previous versions" section.

Make sure you also [read the CHANGELOG](./changelog.html) to see the detailed features and bug-fixes in each version.

## Upgrading to 1.18

Please be advised that license under which Seldon Core is released changes to BSL 1.1 in Seldon Core 1.18.0.
See the LICENSE file in our root folder for more details.

There's no specific work needed for upgrade from 1.17 to 1.18.

## Upgrading to 1.17

There's no specific work needed for upgrade from 1.16 to 1.17.

## Upgrading to 1.16

### Kubernetes version upgrade

Seldon Core 1.16 bumps minimum Kubernetes version to 1.23.
This is because as part of making Seldon Core compatible with Kubernetes 1.25 we moved from `autoscaling/v2beta1` apiVersion of `HorizontalPodAutoscaler` to `autoscaling/v2` (see this [PR](https://github.com/SeldonIO/seldon-core/pull/4172) for further details).

Following table provides a summary of Seldon Core / Kubernetes version compatibility for recent version of Seldon Core.

| Core Version \ K8s Version  | 1.21 | 1.22 | 1.23 | 1.24 | 1.25 | 1.26 | 1.27 |
|-----------------------------|-----:|-----:|-----:|-----:|-----:|-----:|-----:|
|                        1.11 | ✓    |      |      |      |      |      |      |
|                        1.12 | ✓    | ✓    | ✓    | ✓    |      |      |      |
|                        1.13 | ✓    | ✓    | ✓    | ✓    |      |      |      |
|                        1.14 | ✓    | ✓    | ✓    | ✓    |      |      |      |
|                        1.15 | ✓    | ✓    | ✓    | ✓    |      |      |      |
|                        1.16 |      |      | ✓    | ✓    | ✓    | ✓    | ✓    |
|                        1.17 |      |      | ✓    | ✓    | ✓    | ✓    | ✓    |
|                        1.18 |      |      | ✓    | ✓    | ✓    | ✓    | ✓    |

It is always recommended to first upgrade Seldon Core to the latest supported version on your Kubernetes cluster and then upgrade the Kubernetes cluster.

For example, if you using Seldon Core 1.15 on Kubernetes 1.24, it is recommended to **first upgrade Seldon Core to version 1.16** before upgrading your cluster to Kubernetes 1.25 and further.

## Upgrading to 1.15

### Flask upgrade

With this release of Seldon Core we upgrade Flask dependency to 2.x line. This includes Python wrapper and most pre-packaged model servers. In an unlikely scenario that this causes issues one can fallback to `1.14.1` images.

### Services and Deployments Update

The label on services pointing to containers for each node in your inference graph has been changed to stop clashes in key-values for label selectors when you have multiple nodes in your inference graph. This will cause all your existing seldon deployments to be recreated as their service and deployment specs will have changed in the upgrade. **This will cause some downtime in availability**.

## Upgrading to 1.14.1

### OpenShift default storage initializer

In this patch release we fixed the default storage initializer used on OpenShift (both Community and Certified operators).
The image used now is `seldonio/rclone-storage-initializer:1.19.0-dev` which is the same as one used for the non-OpenShift releases (*).

For Certified operator you will find this image defined as
```yaml
- name: RELATED_IMAGE_STORAGE_INITIALIZER
  value: registry.connect.redhat.com/seldonio/rclone-storage-initializer@sha256:a0280c13136dcc870194af72630b9d2f7fc8bcff4edb54dd3bfbce36741af50c
```
in the CSV. This is the same image, just coming from the Red Hat registry.

If you prefer or need to keep using previous storage initializer you can set for Community Operator
```yaml
- name: RELATED_IMAGE_STORAGE_INITIALIZER
  value: kfserving/storage-initializer:v0.6.1
```
or
```yaml
- name: RELATED_IMAGE_STORAGE_INITIALIZER
  value: registry.connect.redhat.com/seldonio/storage-initializer@sha256:52f1e1901fc5de0734515f136847eeb698d48903cf7b6b1cbf273d303bb9029c
```
for the Certified one.

(*) Non-OpenShift releases defaulted to Rclone-based storage initializer in Seldon Core version 1.8.

### Rclone Storage Initializer

Rclone storage initializer image is now based on Red Hat's Universal Base Image (UBI8).
This is not breaking change.


## Upgrading to 1.14

### CRD V1

Only the v1 versions of the CRD will be supported moving forward. The v1beta1 versions will remain in the Helm chart but will be not updated. Allowing the operator to create the CRDs will result in the v1 CRD being created so will only work on Kubernetes clusters >= 1.18.

### Model Health Checks

**Note**: Seldon has adopted the industry-standard Open Inference Protocol (OIP) and is no longer maintaining the Seldon and TensorFlow protocols. This transition allows for greater interoperability among various model serving runtimes, such as MLServer. To learn more about implementing OIP for model serving in Seldon Core 1, see [MLServer](https://docs.seldon.ai/mlserver).

We strongly encourage you to adopt the OIP, which provides seamless integration across diverse model serving runtimes, supports the development of versatile client and benchmarking tools, and ensures a high-performance, consistent, and unified inference experience.
We have updated the health checks done by Seldon for the model nodes in your inference graph. If `executor.fullHealthChecks` is set to true then:
 * For Seldon protocol each node will be probed with `/api/v1.0/health/status`.
 * For tensorflow just TCP checks will be run on the http endpoint.
 * For the Open Inference Protocol (or V2 protocol) each node will be probed with `/v2/health/ready`.

By default we have set `executor.fullHealthChecks` to false for 1.14 release as users would need to rebuild their custom python models if they have not implemented the `health_status` method. In future we will default to `true`.

### Request Logger

The Python request logger component example has been deprecated and removed as part of [#4013](https://github.com/SeldonIO/seldon-core/issues/4013).

### Seldon Core Analytics

In this release we change status of [Seldon Core Analytics](../charts/seldon-core-analytics.html) Helm Chart to example only.
**This chart will not be further updated.** We recommend to install and configure metrics monitoring using [Prometheus Operator](../analytics/analytics.html#metrics-with-prometheus-operator).

## Upgrading to 1.13

### Seldon Inference Payload Logging Changes

The seldon executor logs the prediction request and respose payload to a configured URL post during the inference at each stage of a inference graph. These request/response pairs are now matched correctly to reflect the component's input/output at each node level as compared to pairing based on the tree structure of the inference graph previously. See [relevant issue](https://github.com/SeldonIO/seldon-core/issues/3873) for more details.

### Alibi Detect Server Event Update

The cloud event data that the alibi detect server sends to a configured replyURL post the outlier/drift detection was a JSON string. This is now updated to reflect the content type as JSON correctly.

## Upgrading to 1.12

### Support for Kubernetes 1.22

Seldon Core adds support for Kubernetes 1.22 by upgrading all ValidatingWebhookConfiguration resources to v1 intead of v1beta1. There are no specific breaking steps that were identified when upgrading using the helm charts for each respective version, but you need to make sure to check that this works as expected.


### Updated Python wrapper folder configurations

  * The default running user in Seldon Core is 8888
  * Some servers like the MLFlow Server v1 installs installations in runtime
  * Access required to modify files in the local folder are required so the application folder should be writable
  * The default base image now changes the owner of the /microservice folder to user 8888

### Updated executor request logger settings

The request logging from the executor now has a configurable queue size and write timeout. This will allow a tradeoff between pending request memory usage and failing requests when sending to various logging endpoints that may be slow. The write timeout will mean logging of requests will fail if waiting for more than the given time to be added to the work queue. The two settings are:

  * `executor.requestLogger.workQueueSize` (default 10000)
  * `executor.requestLogger.writeTimeoutMs` (default 2000)

It is also possible to update these values on a per SeldonDeployment basis with the annotations:

 * `seldon.io/executor-logger-queue-size`
 * `seldon.io/executor-logger-write-timeout-ms`

## Upgrading to 1.11

### Python S2I Wrapper

  * The default wrapper `seldonio/seldon-core-s2i-python3` is now Python 3.8
  * Python 3.7 wrapper is still available as `seldonio/seldon-core-s2i-python37`


## Upgrading to 1.10

### Seldon Core Wrapper

 * With introduction of multi-processing in gRPC module the `SO_REUSEPORT` socket option is required. On certain Python distributions you may see `AttributeError: module 'socket' has no attribute 'SO_REUSEPORT'` error which would render gRPC endpoint non-operational. For Anaconda Python distributions we confirmed that upgrading to Python 3.7.10 or 3.8.10 removes the problem.

### Server Updates

 * SKLearn server has been updated to use sklearn 0.24.2
 * XGBoost server has been updated to use xgboost 1.4.2

### Alibi Server Updates

 * Alibi has been updated to 0.6.0
 * Alibi server python has been updated to 3.7.10

## Upgrading to 1.8

### Rclone Storage Initailizer
In Seldon Core 1.8 the rclone-based [storage initializer](https://github.com/SeldonIO/seldon-core/tree/master/components/rclone-storage-initializer) becomes the default one.

The storage initailizer image that is being used is controlled by the helm value:
```yaml
storageInitializer:
  image: seldonio/rclone-storage-initializer:1.19.0-dev
```
and can be customised on per-deployment basis as described in [Prepackaged Model Servers](../servers/overview.md) documentation by setting value of `storageInitializerImage` variable in the graph definition.

This transition requires **creation of the new secrets** for the prepackaged model servers that will be compatible with the rclone configuration format as described [here](../servers/overview.md#handling-credentials). Read more:

- [How to test new secret format](../examples/rclone-upgrade.html)
- [Example cluster upgrade for AWS/MinIO configuration](../examples/global-rclone-upgrade.html)

If you do not wish to configure these secrets now and wish to preserve prior behaviour you can opt for usage of previous storage initializer by using following helm value:
```yaml
storageInitializer:
  image: kfserving/storage-initializer:v0.6.1
```
See further documentation [here](../servers/kfserving-storage-initializer.md).


### Request Logger

In Seldon Core 1.9 we will be moving [seldon-request-logger](https://github.com/SeldonIO/seldon-core/tree/v1.8.0/components/seldon-request-logger) to separate repository.


### Legacy Java Engine Orchestrator

In Seldon Core 1.9 final deprecation of Java Engine will happen with removal of all the related code from the repository.


## Upgrading to 1.7

### Python Dependency Updates

Various CVEs were resolved via #2970, which included several packages upgrades which may affect applications that install packages which may not be compatible. This also includes the installation of pip==20.2, however this version of pip still uses the older resolver.

## Upgrading to 1.6

### Webhook Removal

As part of the 1.6.0 release we are removing the Seldon Core Mutating Webhook. This won't cause any noticeable changes, but it is recommended that you manually remove the webhook once you upgrade to version 1.6.0


## Upgrading to 1.5

### REST and gRPC

To take advantage of the ability to handle both REST and gRPC on any deployed model python model images will need to be recreated using the 1.5 python wrapper. If they are not updated they will only expose the protocol they were originally wrapped for.

You can use and extend the [backwards compatibility notebook](../examples/backwards_compatibility.html) to check your deployments will work if you do not intend to upgrade them.

## Upgrading to Kubernetes version >= 1.18

If you have a Kubernetes cluster with Seldon Core installed, and you want to upgrade the Kubernetes cluster, you have to carry out a set of manual steps due to the more strict validation that this version of Kubernetes introduced.

To be more specific, we had to provide two versions of the CRD as part of the seldon core install helm chart. Similarly the CRD for Seldon Core in Kubernetes post-1.18 is actually differnt - namely, you can actually see that in version 1.3.0 we introduced new CRD changes in the helm chart via an IF statement to use a different CRD depending on the k8s version. Due to this, the path to upgrade from pre-1.18 to post-1.18 requires the following manual steps to be carried out:

1. Start with kubernetes cluster pre 1.18 with seldon core pre-1.3.0
2. Upgrade Kubernetes cluster to post 1.18 (seldon core CRD is now "invalid" but still installed as still in etcd)
3. Manually add "spec.preserveUnknownFields", to helm chart and install CRD (so it ignores invalid fields of now invalid CRD)
4. Remove the "spec.preserveUnknownFields", from helm chart manually again, and re-install now the current CRD

## Upgrading to 1.3

### Breaking Changes

The version of sklearn used by the default sklearn server will be 0.23.2. To use a different version you will need to follow the steps described in the [sklearn server documentation](../servers/sklearn.html).

## Upgrading to 1.2.1

*[NOTE]* 1.2.0 has issue where all Seldon Deployments are marked as "NotReady" as there is a [bug caused by a volumeName update](https://github.com/SeldonIO/seldon-core/issues/2017). This can be resolved by following the 1.2.0 volume patch [as outlined by this example](../examples/patch_1_2.html). It is recommended to upgrade to version 1.2.1 directly instead.

All seldon-managed pods will be subject to a rolling update as part of this upgrade.

### New Features / Breaking Changes

 * The helm value `createResources` has been renamed `managerCreateResources`.
 * To allow CRDs to be created by the manager. If `managerCreateResources` is true then extra RBAC to `create` CRDs is added from the previous versions RBAC which was to just list and get.
 * If upgrading the analytics helm chart then a `kubectl delete deployment -n seldon-system -l app=grafana` should be [run first](https://github.com/SeldonIO/seldon-core/pull/1917)
 * All the prepackaged model servers are now created with RedHat UBI images. One consequence of this is that they will all run as non-root as it best practice.

### Request Logger

The values.yaml for the seldon-core-operator helm chart has changed. The field `defaultRequestLoggerEndpointPrefix` is replaced by:

```yaml
  requestLogger:
    defaultEndpoint: 'http://default-broker'
```

This default value will find a broker in the model's namespace. To point to another namespace it would be `default-broker.anothernamespace`.

## Upgrading to 1.1 from previous versions

As we moved to 1.x+ there are several breaking changes that need to be considered. These are outlined below

### New Features / Breaking Changes

#### Deployment Naming and Rolling Updates

The deployments created by Seldon Core have been changed to follow a fixed scheme. It will now be:

```text
<seldondeployment name>-<predictor name>-<podspec idx>-<container names>
```

So for example:

```yaml
apiVersion: machinelearning.seldon.io/v1
kind: SeldonDeployment
metadata:
  name: rest-seldon
spec:
  name: restseldon
  protocol: seldon
  transport: rest
  predictors:
  - componentSpecs:
    - spec:
        containers:
        - image: seldonio/mock_classifier_rest:1.3
          name: classifier
    graph:
      name: classifier
      type: MODEL
    name: model
    replicas: 1
```

For the above resource, one Deployment will be created with name:

```text
rest-seldon-model-0-classifier
```

This will change how rolling updates are done. Now any change to the first PodSpec above will be updated via a rolling update as expected if the names of the containers are not changed. If however you changed "classifier" to "classifier2" you would get a new deployment created which would replace the old deployment when running.


#### New Service Orchestrator

From version 1.1 Seldon Core comes with a new service orchestrator written in Go which replaces the previous Java engine. Some breaking changes are present:

 * Metadata fields in the Seldon Protocol are no longer added. Any custom metata data will need to be added and exposed to Prometheus metrics by the individual components in the graph
 * All components in the graph must either be REST or gRPC and only the given protocol is exposed externally.
 * The metric names placed in Prometheus have changed to include the `executor` name rather than `engine` : see the [analytics docs](../analytics/analytics.html)

The new service orchestrator comes with several advantages including ability to handle Tensorflow REST and gRPC protocols and full metrics and tracing support for both REST and gRPC.

For those wishing to use the deprecated Java engine service orchestrator see [the service orchestrator docs](../graph/svcorch.md) for details.

#### Ambassador Retries

Ambassador retries has been removed from the previous hardwired value of 3. Retries is now available via an [annotation for Ambassador](../ingress/ambassador.html).


### Python Wrapper Tag Update

The Python Wrapper was using naming convention in the format 0.1 ... 0.18. In this release we have renamed the version of the Python Wrapper tag to match the same convention as the Executor, Operator, etc. This means that the Python Wrapper tag for this release is 1.1, and the snapshot would be 1.1.1-SNAPSHOT

### Dated SNAPSHOTS

Whenever a new PR was merged to master, we have set up our CI to build a "SNAPSHOT" version, which would contain the Docker images for that specific development / master-branch code.

Previously, we always had the SNAPSHOT tag being overridden with the latest. This didn't allow us to know what version someone may be trying out when using master, so we wanted to introduce a way to actually get unique tags for every image that gets landed into master.

Now every time that a PR is landed to master, a new "dated" SNAPSHOT version is created, which pushes images with the tag `"<next-version>-SNAPSHOT_<timestamp>"`. A new branch is also created with the name `"v<next-version>-SNAPSHOT_<timestamp>"`, which contains the respective helm charts, and allows for the specific version (as outlined by the version in `version.txt`) to be installed.

You can follow the instructions in the [installation page](../workflow/install.md) to install the snapshot version.

### Wrapper compatibility table

To verify if Seldon Core v1.0 and v.1.1 is compatible with older s2i wrapper versions we conducted a simple test with a one-node model.
The model has been deployed both with REST and GRPC API with both new orchestrator and the deprecated Java engine (v1.0 only with Java Engine).
Test verifies if model can successfully serve inference requests.

**NOTE:** Full support of custom metrics and tags with new orchestrator is only available from Python wrapper version 0.19.
If you need to use older version of Python wrapper you can continue to use Java engine as described above until the next release.


| Language Wrapper |     Version   | API Type | New Orchestrator  | Deprecated Java engine | Notes                                   |
|------------------|---------------|----------|-------------------|------------------------|-----------------------------------------|
| Python           | 0.19          | both     | yes               | yes                    | full support of custom metrics and tags |
| Python           | 0.11 ... 0.18 | both     | yes               | yes                    |                .                        |
| Python           | 0.10          | REST     | no                | yes                    |                .                        |
| Python           | 0.10          | GRPC     | yes               | yes                    |                .                        |
| Python           | < 0.10        | GRPC     | ?                 | ?                      |                .                        |
| Java             | 0.2 & 0.1     | REST     | yes               | yes                    |   minor difference in request format    |
| Java             | 0.2 & 0.1     | GRPC     | yes               | yes                    |                .                        |



Example of request format difference with Java wrapper deployed with REST API:

1. Using new orchestrator:
```bash
curl -s -X POST \
    -d 'json={"data": {"names": ["a", "b"], "ndarray": [[1.0, 2.0]]}}' \
    localhost:8003/seldon/seldon/compat-rest-java-02-executor/api/v1.0/predictions
```

2. Using deprecated Java engine:
```bash
curl -s -X POST -H 'Content-Type: application/json' \
    -d '{"data": {"names": ["a", "b"], "ndarray": [[1.0, 2.0]]}}' \
    localhost:8003/seldon/seldon/compat-rest-java-02-engine/api/v1.0/predictions
```


## Upgrading to 0.5.2 from previous versions

This version included significant improvements and features, including the addition of pre-packaged model servers, fixing several critical bugs.

This was the version that also dropped support for kubernetes 1.11, and added changes in the mutating and validating webhooks that you need to make sure are transitioned as outlined below.

### Upgrading process

In order to upgrade, the main requirement is to make sure that the Kubernetes cluster is updated to 1.12 or higher.

Once this is done, it's necessary to delete the old webhooks. This can be done with the following commands (you need to make sure that the commands are executed in the namespace in which seldon core was installed).

## Upgrading to 0.2.8 from previous versions.

### Upgrading process

#### Installation process now with Helm

The helm charts to install Seldon Core have changed. There is now a single Helm chart `seldon-core-operator` that installs the CRD and its controller. Ingress options are now separate and you need to choose between the available options which are at present:

 * Ambassador - via its official Helm chart
 * Istio

For more details see the [install docs](../workflow/install.md).

The Helm chart `seldon-core-operator` will require clusterwide RBAC and should be installed by a cluster admin.

##### Dropping support for KSonnet

Ksonnet is now deprecated. You should convert to using Helm to install Seldon Core.
