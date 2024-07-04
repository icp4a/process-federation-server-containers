# Defining a federated data repository for Process Federation Server containers

Process Federation Server uses a [remote federated data respository](https://www.ibm.com/docs/en/baw/23.x?topic=service-declaring-federated-data-repository-in-serverxml), implemented with an Elasticsearch or Opensearch cluster, to store data.

It supports the following products and versions for federated data repository:
* All Elasticsearch 7.x versions from 7.17.0 (deprecated)
* All Elasticsearch 8.x versions
* All Opensearch 2.x versions from 2.5.0

> Note: the support of Elasticsearch 7.x is deprecated and might be removed in a future version of Process Federation Server containers.

Depending if you are installing a stand-alone IBM® Business Automation Workflow on containers or an IBM Business Automation Workflow Server (or Workflow Runtime) in the IBM Cloud Pak for Business Automation, your options for the federated data repository implementation differs.

If you are installing a __stand-alone IBM Business Automation Workflow on containers__, the options for the federated data repository are:

* __standalone-BAW-FDR:__ let stand-alone IBM Business Automation Workflow on containers create a statefulset which runs Opensearch. This is the default option on AMD64 architectures. This option is not supported on other architectures.

* __External-FDR:__ provide your own Elasticsearch or Opensearch cluster separately and reference it in the ProcessFederationServer Custom Resource

If you are installing __IBM Cloud Pak for Business Automation__, the options for the federated data repository are:

* __CPfs-OS:__ let the IBM Cloud Pak for Business Automation provision and use Opensearch provided by IBM® Cloud Pak Foundational Services (CPfs). This is the default option.

* __External-FDR:__ provide your own Elasticsearch or Opensearch cluster separately and reference it in the ProcessFederationServer Custom Resource. This option is only available if `spec.shared_configuration.sc_deployment_type` is set to "Production".

> Note: The deployment type is determined by the spec.shared_configuration.sc_deployment_type property of the ICP4ACluster Custom Resource which can be set to either Starter or Production

* **[Using Opensearch provided by IBM Cloud Pak foundational services](./Using-CPfs-Opensearch.md)**

  When deploying the IBM Cloud Pak for Business Automation, by default, the ICP4ACluster Custom Resource is configured to provision and use Opensearch provided by IBM Cloud Pak foundational services.
  
* **[Deploying Opensearch as part of the stand-alone IBM Business Automation Workflow on containers deployment](./Using-standalone-BAW-Opensearch.md)**

  When deploying stand-alone IBM Business Automation Workflow on containers on AMD64 architectures, by default the ProcessFederationServer Custom Resource is configured to create a [statefulset](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/), `<cr-instance-name>-elasticsearch-statefulset`, that deploys pods running Opensearch. This option is not supported on other architectures.

* **[Referencing your own Elasticsearch or Opensearch cluster](./Using-own-Elasticsearch-or-Opensearch.md)**

  Instead of using one the Elasticsearch services that can be automatically deployed as part of the stand-alone IBM Business Automation Workflow on containers deployment or the IBM Business Automation Workflow Server (or Workflow Runtime) in the IBM Cloud Pak for Business Automation deployment, you can decide to reference your own Elasticsearch or Opensearch cluster.

---

**Parent topic:** [Architecture of a Process Federation Server container environment](./Architecture.md)

**Index:** [Documentation index](../README.md#documentation-index)
