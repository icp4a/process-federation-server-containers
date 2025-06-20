# Referencing your own Elasticsearch or Opensearch cluster

Instead of using Elasticsearch or Opensearch services that can be automatically deployed as part of the stand-alone IBM Business Automation Workflow on containers deployment or the IBM Cloud Pak for Business Automation deployment, you can decide to reference your own Elasticsearch or Opensearch cluster.

If you want to use your own Elasticsearch or Opensearch then you have to declare it in the IBM Process Federation Server configuration as well as in the configuration of the federated systems running on containers in the same namespace as IBM Process Federation Server. Here are the links to the doc for each of this system
- [IBM Process Federation Server](https://www.ibm.com/docs/baw/25.0.0?topic=containers-referencing-external-federated-data-repository)
- [IBM Business Automation Workflow](https://www.ibm.com/docs/cloud-paks/cp-biz-automation/25.0.0?topic=customizing-enabling-full-text-search)
- [IBM Workflow Process Service](https://www.ibm.com/docs/cloud-paks/cp-biz-automation/25.0.0?topic=deployment-customizing-workflow-process-service-runtime#tsk_customize_wfps__pfs__title__1)

> Note: automatic Elasticsearch and Opensearch index creation is controlled by the `action.auto_create_index` setting. By default, this setting is set to true, which allows any index to be created automatically. The remote Elasticsearch or Opensearch cluster used by IBM Process Federation Server must be configured to prevent automatic index creation. The documentation of this setting for Elasticsearch can be found here: https://www.elastic.co/guide/en/elasticsearch/reference/8.4/docs-index_.html#index-creation. For Opensearch, the documentation can be found here: https://docs.opensearch.org/docs/latest/install-and-configure/configuring-opensearch/index-settings/

---

**Parent topic:** [Defining a Federated Data Repository for IBM Process Federation Server containers](./Defining-a-federated-data-repository.md)

**Index:** [Documentation index](../README.md#documentation-index)
