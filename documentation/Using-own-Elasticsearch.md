# Referencing your own Elasticsearch

Instead of using Elasticsearch as part of the stand-alone IBM® Business Automation Workflow on containers deployment or the IBM Business Automation Workflow Server (or Workflow Runtime) in the IBM Cloud Pak for Business Automation deployment, you can decide to reference your own Elasticsearch.

You can reference it with the following CR values:
* [pfs_configuration.elasticsearch.endpoint](https://www.ibm.com/docs/en/baw/20.x?topic=workflow-business-automation-server-parameters#ref_baw_params__pfs): the URL to access your Elasticsearch service
* [pfs_configuration.elasticsearch.admin_secret_name](https://www.ibm.com/docs/en/baw/20.x?topic=workflow-business-automation-server-parameters#ref_baw_params__pfs): a [secret](https://kubernetes.io/fr/docs/concepts/configuration/secret/) containing the username and password that Process Federation Server will use to access Elasticsearch. The keys expected in this [secret](https://kubernetes.io/fr/docs/concepts/configuration/secret/) are respectively username and password.

The following example shows the parameters to set:

```
pfs_configuration:
      elasticsearch:
         endpoint: https://es_host:9201
         admin_secret_name: demo-elasticsearch-admin-secret
         connect_timeout: 10s
         read_timeout: 30s
         thread_count: 0
         max_connection_total: -1
         max_connection_per_route: -1
```

> Note: Elasticsearch automatic index creation is controlled by the action.auto_create_index setting. By default, this setting is set to true, which allows any index to be created automatically. The remote Elasticsearch cluster used by Process Federation Server must be configured to prevent automatic index creation. The documentation of this setting can be found here: https://www.elastic.co/guide/en/elasticsearch/reference/7.x/docs-index_.html#index-creation

---

**Parent topic:** [Using Elasticsearch when running Process Federation Server with IBM Cloud Pak for Business Automation](./Using-Elasticsearch.md)

**Index:** [Documentation index](../README.md#documentation-index)
