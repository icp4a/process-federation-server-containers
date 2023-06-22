# Referencing your own Elasticsearch or Opensearch cluster

Instead of using one the Elasticsearch services that can be automatically deployed as part of the stand-alone IBM Business Automation Workflow on containers deployment or the IBM Cloud Pak for Business Automation deployment, you can decide to reference your own Elasticsearch or Opensearch cluster.

You can reference it with the following CR values:
* [pfs_configuration.elasticsearch.endpoint](https://www.ibm.com/docs/en/cloud-paks/cp-biz-automation/22.0.1?topic=parameters-business-automation-workflow-runtime-workstream-services#ref_baw_params__pfserver): the URLs to access your Elasticsearch or Opensearch cluster
* [pfs_configuration.elasticsearch.admin_secret_name](https://www.ibm.com/docs/en/cloud-paks/cp-biz-automation/22.0.1?topic=parameters-business-automation-workflow-runtime-workstream-services#ref_baw_params__pfserver): a [secret](https://kubernetes.io/fr/docs/concepts/configuration/secret/) containing the username and password that Process Federation Server will use to access the cluster. The keys expected in this [secret](https://kubernetes.io/fr/docs/concepts/configuration/secret/) are respectively username and password.
* If the Elasticsearch or Opensearch cluster exposes self-signed certificates, these certificates must be added to the Process Federation Server truststore. For each of these certificates, you must create a [secret](https://kubernetes.io/docs/concepts/configuration/secret/) resource with an entry named `tls.crt` which contains the public key of the certificate in PEM format. Once all the self-signed certificates are packaged into secrets, they must be declared under the [pfs_configuration.tls.tls_trust_list](https://www.ibm.com/docs/en/cloud-paks/cp-biz-automation/22.0.1?topic=parameters-business-automation-workflow-runtime-workstream-services#ref_baw_params__pfserver) CR value.

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

> Note: automatic Elasticsearch and Opensearch index creation is controlled by the _action.auto_create_index_ setting. By default, this setting is set to true, which allows any index to be created automatically. The remote Elasticsearch or Opensearch cluster used by Process Federation Server must be configured to prevent automatic index creation. The documentation of this setting for Elasticsearch can be found here: https://www.elastic.co/guide/en/elasticsearch/reference/8.4/docs-index_.html#index-creation. For Opensearch, the documentation can be found here: https://opensearch.org/docs/latest/api-reference/cluster-api/cluster-settings/

---

**Parent topic:** [Defining a federated data repository for Process Federation Server containers](./Defining-a-federated-data-repository.md)

**Index:** [Documentation index](../README.md#documentation-index)
