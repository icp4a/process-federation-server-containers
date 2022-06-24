# Deploying Elasticsearch as part of the stand-alone IBM Business Automation Workflow on containers deployment

When deploying stand-alone IBM® Business Automation Workflow on containers on Intel architectures (AMD64 or x86_64 the 64-bit edition for Linux x86), by default the ICP4ACluster Custom Resource is configured to create a [statefulset](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/), `<icp4acluster-instance-name>-elasticsearch-statefulset`, that deploys pods running Elasticsearch. This option is not supported on IBM Z (s390x) architectures.

Each pod runs:
* an init container (`ssl-init-container`) which creates keystore and truststore
* an optional privileged container (`sysctl`) which updates the worker node executing `sysctl` commands as root user. For more details, see section [Privileged init container](#privileged-init-container)
* a container (`elasticsearch`) running Elasticsearch
* a container (`nginx`) running Nginx to provide basic authentication and HTTPS support in front of the Elasticsearch REST API.

See [IBM Business Automation Workflow on containers parameters](https://www.ibm.com/docs/en/baw/20.x?topic=workflow-parameters) for the list of values needed to configure Elasticsearch in Process Federation Server.

## Privileged init container

Elasticsearch pods require the hosting worker nodes to be configured to:
* [disable memory swapping](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/setup-configuration-memory.html) by setting the **sysctl** value `vm.swappiness` to `1`
* [increase the limit on the number of open files descriptors](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/file-descriptors.html) for the user that runs Elasticsearch by setting **sysctl** value `vm.max_map_count` to `65,536` or higher.
* or set up Elasticsearch on your own, separately, as explained in [Referencing your own Elasticsearch](./Using-own-Elasticsearch.md).

By default, Elasticsearch pods create a privileged init-container that runs the following commands as root user:

```
sysctl -w vm.max_map_count=262144 && sed -i '/^vm.max_map_count /d' /etc/sysctl.conf && echo 'vm.max_map_count = 262144' >> /etc/sysctl.conf && sysctl -w vm.swappiness=1 && sed -i '/^vm.swappiness /d' /etc/sysctl.conf && echo 'vm.swappiness=1' >> /etc/sysctl.conf
```

This enables the pod to apply the [related mandatory configuration settings](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/system-config.html), increasing the virtual memory and disabling swapping on the worker node.
If you cannot run privileged container, you can:
* either perform the above commands manually on all worker nodes and set [elasticsearch_configuration.privileged](https://www.ibm.com/docs/en/baw/20.x?topic=workflow-business-automation-server-parameters#ref_baw_params__elastic) to `false`
* or set up Elasticsearch on your own, separately from IBM Cloud Pak for Business Automation, as explained in [Referencing your own Elasticsearch](./Using-own-Elasticsearch.md).

## Managing users

By default, a single user is created to access the Elasticsearch REST API. This user is referenced in the [secret](https://kubernetes.io/fr/docs/concepts/configuration/secret/) named `<icp4acluster-instance-name>-elasticsearch-admin-secret`. This [secret](https://kubernetes.io/fr/docs/concepts/configuration/secret/) contains the following keys:

* `.htpasswd`: [a file listing the users and associated passwords](https://docs.nginx.com/nginx/admin-guide/security-controls/configuring-http-basic-authentication/#pass) that are allowed to authenticate.
* `username`: the name of a user referenced in the `.htpasswd` file. Process Federation Server pods use this user to connect to Elasticsearch.
* `password`: the password of the user provided through the `username` key. Process Federation Server pods use this password to connect to Elasticsearch.

In production, you must provide your own set of users by creating a similar secret, and reference it under the [elasticsearch_configuration.admin_secret_name](https://www.ibm.com/docs/en/baw/20.x?topic=workflow-business-automation-server-parameters#ref_baw_params__elastic) CR value.

## Elasticsearch data

By default, each Elasticsearch pod persists its data in a dedicated persistent volume. You can customize the persistence by setting [elasticsearch_configuration.storage.*](https://www.ibm.com/docs/en/baw/20.x?topic=workflow-business-automation-server-parameters#ref_baw_params__elastic) CR values.

## Elasticsearch snapshots

To be able to use the [snapshot API of Elasticsearch](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/modules-snapshots.html), you must provide a persistent volume that will be used by all pods. You can enable snapshot storage persistent volume by defining the [elasticsearch_configuration.snapshot_storage.*](https://www.ibm.com/docs/en/baw/20.x?topic=workflow-business-automation-server-parameters#ref_baw_params__elastic) CR values.

The persistent volume is locally mounted on pods at: `/mnt/snapshots`.

## Exposing Elasticsearch service using a Route

The ICP4ACluster Custom Resource does not expose the Elasticsearch service outside of the Kubernetes cluster.

If you need to externally expose the Elasticsearch REST API and use your own tooling to monitor or administer Elasticsearch, you can:

* either change the [elasticsearch_configuration.service_type](https://www.ibm.com/docs/en/baw/20.x?topic=workflow-business-automation-server-parameters#ref_baw_params__elastic) CR value to `NodePort`
* or, if running on OpenShift, create a secured Route which will expose port 9201 of the `<icp4acluster-instance-name>-elasticsearch-service`

---

**Parent topic:** [Using Elasticsearch when running Process Federation Server with IBM Cloud Pak for Business Automation](./Using-Elasticsearch.md)

**Index:** [Documentation index](../README.md#documentation-index)
