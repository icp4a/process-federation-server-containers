# Defining a Federated Data Repository for IBM Process Federation Server containers

IBM Process Federation Server uses a [remote Federated Data Repository](https://www.ibm.com/docs/baw/25.0.0?topic=service-declaring-federated-data-repository-in-serverxml), which can be an Elasticsearch or Opensearch cluster.

It supports the following products and versions for Federated Data Repository:
* All Elasticsearch 7.x versions from 7.17.0 (deprecated)
* All Elasticsearch 8.x versions
* All Opensearch 2.x versions from 2.19.x

> Note: the support of Elasticsearch 7.x is deprecated and might be removed in a future version of IBM Process Federation Server containers.

For each federated system an index will be created in the Federated Data Repository where documents related to tasks, process instances and/or case instances will be indexed. These documents can be indexed directly by IBM Business Automation Workflow or by IBM Process Federation Server. And IBM Process Federation Server REST API will search into this indexes to return federated lists of tasks, process instances and/or case instances.

The options for the Federated Data Repository are:

* **[Using Opensearch provided by IBM Cloud Pak foundational services](./Using-CPfs-Opensearch.md)**

  When deploying the IBM Cloud Pak for Business Automation or IBM Business Automation Workflow on Container, Opensearch provided by IBM Cloud Pak foundational services will be installed depending on the selected capability, pattern, and optional components.
  
* **[Referencing your own Elasticsearch or Opensearch cluster](./Using-own-Elasticsearch-or-Opensearch.md)**

  Instead of using the Opensearch provided by IBM Cloud Pak foundational services, you can choose to reference your own Elasticsearch or Opensearch cluster.

---

**Parent topic:** [Architecture of an IBM Process Federation Server container environment](./Architecture.md)

**Index:** [Documentation index](../README.md#documentation-index)
