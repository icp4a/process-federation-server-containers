# Architecture of an IBM Process Federation Server container environment

  The IBM Process Federation Server operator creates several kubernetes resources to deploy IBM Process Federation Server. The topics in this section provide detailed information about those resources.
  
  * **[IBM Process Federation Server statefulset](./PFS-Statefulset.md)**

    IBM Process Federation Server is deployed as a [statefulset](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/) named `<cr-instance-name>-pfs`. This section provide detailed information about this statefulset, how to access its REST API and how to configure it.

  * **[Defining a Federated Data Repository for IBM Process Federation Server containers](./Defining-a-federated-data-repository.md)**

    IBM Process Federation Server uses a [remote Federated Data Repository](https://www.ibm.com/docs/baw/25.0.0?topic=server-process-federation-components-overview), implemented with an Elasticsearch or Opensearch cluster. The topics in this section provides detailed information about the different types of Elasticsearch or Opensearch clusters that can be used by IBM Process Federation Server containers to implement the Federated Data Repository:

    * **[Using an Opensearch cluster provided by IBM Cloud Pak foundational services](./Using-CPfs-Opensearch.md)**

      When deploying IBM Cloud Pak for Business Automation or stand-alone IBM Business Automation Workflow on containers, an Opensearch cluster provided by IBM Cloud Pak foundational services can provisioned. This section provides information about using this type of Opensearch cluster as the IBM Process Federation Server Federated Data Repository.

    * **[Referencing your own Elasticsearch or Opensearch cluster](./Using-own-Elasticsearch-or-Opensearch.md)**

      You can reference your own Elasticsearch or Opensearch cluster to use it as the remote Federated Data Repository of your IBM Process Federation Server deployment. This section provide information about referencing this type of clusters.
---
**Parent topic:** [Administering and operating IBM Process Federation Server containers](../README.md)

**Index:** [Documentation index](../README.md#documentation-index)
