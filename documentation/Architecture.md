# Architecture of a Process Federation Server container environment

When Process Federation Server is deployed using an ICP4ACluster Custom Resource, several kubernetes resources are created. The topics in this section provide detailed information about those resources.

* **[Process Federation Server statefulset](./PFS-Statefulset.md)**

    Process Federation Server is deployed as a [statefulset](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/) named `<icp4acluster-instance-name>-pfs`.

* **[Using Elasticsearch when running Process Federation Server with IBM Cloud Pak for Business Automation](./Using-Elasticsearch.md)**

    Process Federation Server uses a [remote Elasticsearch service](https://www.ibm.com/docs/en/baw/20.x?topic=service-configuring-remote-elasticsearch) to store data.


---
**Parent topic:** [Administering and operating IBM Process Federation Server Containers](../README.md)

**Index:** [Documentation index](../README.md#documentation-index)
