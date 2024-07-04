# Using Opensearch provided by IBM Cloud Pak foundational services

When deploying the IBM® Cloud Pak for Business Automation, by default, the ICP4ACluster Custom Resource is configured to provision and use Opensearch provided by IBM Cloud Pak foundational services.

IBM Cloud Pak foundational services will deploy a [statefulset](https://kubernetes.io/fr/docs/concepts/workloads/controllers/statefulset/) named `opensearch-ib-6fb9-es-server-all` with the following number of replicas:

* 3 replica if `spec.shared_configuration.sc_deployment_type` property of the ICP4ACluster Custom Resource is set to "starter" or if `spec.shared_configuration.sc_deployment_profile_size` is set to "small" or "medium";

* 3 replicas if `spec.shared_configuration.sc_deployment_type` property of the ICP4ACluster Custom Resource is set to "production" and `spec.shared_configuration.sc_deployment_profile_size` is set to "large".

## Accessing the Opensearch cluster REST API

To access the Opensearch REST API, you must retrieve its route and credentials by executing the command:

```
oc get elasticsearchcluster opensearch -o yaml
```

Opensearch access information is in the returned content:
* `spec.credentialSecret`(where default value is `opensearch-ibm-elasticsearch-cred-secret` if `spec.credentialSecret` is not set. The secret key is the username, and secret key value is the password.)
* `status.url`

This is a sample output:

```
kind: ElasticsearchCluster
spec:
  credentialSecret: opensearch-cred-secret
status:
  phase: Ready
  url: 'https://opensearch-ibm-elasticsearch-srv'
```

Once you have the service name, you can now identify the Route URL which externally exposes the service retrieved in the status of the elasticsearchcluster custom resource YAML:

```
oc get route | grep opensearch-ibm-elasticsearch-srv
```

The second field is the hostname to use to access the Opensearch REST API. You can now access Opensearch using a URL to this hostname and the credentials retrieved previously:

```
https://opensearch.myproject.myopenshift.com
```



---

**Parent topic:** [Defining a federated data repository for Process Federation Server containers](./Defining-a-federated-data-repository.md)

**Index:** [Documentation index](../README.md#documentation-index)
