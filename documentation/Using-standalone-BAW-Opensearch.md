# Deploying Opensearch as part of the stand-alone IBM Business Automation Workflow on containers deployment

When deploying stand-alone IBM® Business Automation Workflow on containers on AMD64 architectures, by default the ProcessFederationServer Custom Resource is configured to create a [statefulset](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/), `<cr-instance-name>-elasticsearch-statefulset`, that deploys pods running Opensearch. This option is not supported on other architectures.

> **Note:** The stand-alone IBM Business Automation Workflow on containers uses an OpenSearch image instead of Elasticsearch since 23.0.2. The kubernetes resources' names do not have changes. For example, the Opensearch statefulset still names `<cr-instance-name>-elasticsearch-statefulset`.

Each pod runs:
* a container (`elasticsearch`) running Opensearch
* an init container (`ssl-init-container`) which creates keystore and truststore
* an optional privileged container (`sysctl`) which updates the worker node executing `sysctl` commands as root user. For more details, see section [Privileged init container](#privileged-init-container)
* an init container (`es-folder-prepare`) that prepares the opensearch container filesystem for readonly operations

See [IBM Business Automation Workflow on containers parameters](https://www.ibm.com/docs/en/baw/23.x?topic=workflow-parameters) for the list of values needed to configure Opensearch in Process Federation Server.

## Privileged init container

Opensearch pods require the hosting worker nodes to be configured to:
* [disable memory swapping](https://www.elastic.co/guide/en/elasticsearch/reference/8.4/setup-configuration-memory.html) by setting the **sysctl** value `vm.swappiness` to `1`
* [increase the limit on the number of open files descriptors](https://www.elastic.co/guide/en/elasticsearch/reference/8.4/file-descriptors.html) for the user that runs Opensearch by setting **sysctl** value `vm.max_map_count` to `65,536` or higher.
* or set up Opensearch on your own, separately, as explained in [Referencing your own Elasticsearch or Opensearch cluster](./Using-own-Elasticsearch-or-Opensearch.md).

By default, Opensearch pods create a privileged init-container that runs the following commands as root user:

```
sysctl -w vm.max_map_count=262144 && sysctl -w vm.swappiness=1
```

This enables the pod to apply the [related mandatory configuration settings](https://www.elastic.co/guide/en/elasticsearch/reference/8.4/system-config.html), increasing the virtual memory and disabling swapping on the worker node.
If you cannot run privileged container, you can:
* either perform the above commands manually on all worker nodes and set [elasticsearch_configuration.privileged](https://www.ibm.com/docs/en/baw/23.x?topic=workflow-parameters) to `false`
* or set up Opensearch on your own, separately from IBM Cloud Pak for Business Automation, as explained in [Referencing your own Elasticsearch or Opensearch cluster](./Using-own-Elasticsearch-or-Opensearch.md).

## Managing users

By default, a single user is created to access the Opensearch REST API. This user is referenced in the [secret](https://kubernetes.io/fr/docs/concepts/configuration/secret/) named `<cr-instance-name>-elasticsearch-admin-secret`. This [secret](https://kubernetes.io/fr/docs/concepts/configuration/secret/) contains the following keys:

* `username`: the name of a user referenced in the `sensitive` file. Process Federation Server pods use this user to connect to Opensearch.
* `password`: the password of the user provided through the `username` key. Process Federation Server pods use this password to connect to Opensearch.
* `sensitive`: a file containing the user and associated passwords that is mount to Opensearch container.


In production, you must provide your own set of users by creating a similar secret, and reference it under the [elasticsearch_configuration.admin_secret_name](https://www.ibm.com/docs/en/baw/23.x?topic=workflow-parameters) CR value.

## Opensearch data

By default, each Opensearch pod persists its data in a dedicated persistent volume. You can customize the persistence by setting [elasticsearch_configuration.storage.*](https://www.ibm.com/docs/en/baw/23.x?topic=workflow-parameters) CR values.

_Note: In production, the persistent volumes storing Elasticsearch data should be configured to use `block` storage rather than `file` storage. Follow this [link](https://www.ibm.com/cloud/blog/object-vs-file-vs-block-storage) for more details about `block` and `file` storage._

## Opensearch snapshots

To be able to use the [snapshot API of Opensearch](https://opensearch.org/docs/latest/api-reference/snapshots/index/), you must provide a persistent volume that will be used by all pods. You can enable snapshot storage persistent volume by defining the [elasticsearch_configuration.snapshot_storage.*](https://www.ibm.com/docs/en/baw/23.x?topic=workflow-parameters) CR values.

The persistent volume is locally mounted on pods at: `/mnt/snapshots`.

## Exposing Opensearch service using a Route

The ProcessFederationServer Custom Resource does not expose the Opensearch service outside of the Kubernetes cluster.

If you need to externally expose the Opensearch REST API and use your own tooling to monitor or administer Opensearch, you can:

* either change the [elasticsearch_configuration.service_type](https://www.ibm.com/docs/en/baw/23.x?topic=workflow-parameters) CR value to `NodePort`
* or, if running on OpenShift, create a secured Route which will expose port 9201 of the `<cr-instance-name>-elasticsearch-service`

---

**Parent topic:** [Defining a federated data repository for Process Federation Server containers](./Defining-a-federated-data-repository.md)

**Index:** [Documentation index](../README.md#documentation-index)
