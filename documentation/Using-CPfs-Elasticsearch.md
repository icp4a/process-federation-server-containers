# Using Elasticsearch provided by IBM Cloud Pak foundational services

When deploying the IBM® Cloud Pak for Business Automation, by default, the ICP4ACluster Custom Resource is configured to provision and use Elasticsearch provided by IBM Cloud Pak foundational services.

IBM Cloud Pak foundational services will deploy a [statefulset](https://kubernetes.io/fr/docs/concepts/workloads/controllers/statefulset/) named `iaf-system-elasticsearch-es-data` with the following number of replicas:

* 1 replica if `spec.shared_configuration.sc_deployment_type` property of the ICP4ACluster Custom Resource is set to "starter" or if `spec.shared_configuration.sc_deployment_profile_size` is set to "small" or "medium";

* 3 replicas if `spec.shared_configuration.sc_deployment_type` property of the ICP4ACluster Custom Resource is set to "production" and `spec.shared_configuration.sc_deployment_profile_size` is set to "large".

## Accessing the Elasticsearch cluster REST API

To access the Elasticsearch REST API, you must retrieve its route and credentials by executing the command:

```
oc get cartridgerequirements icp4ba -o yaml
```

Elasticsearch access information is in the returned content:
* `status.components.elasticsearch.endpoints[x].uri` (where `status.components.elasticsearch.endpoints[x].scope` is `External`)
* `status.components.elasticsearch.endpoints[x].authentication.secret.secretName`(where `status.components.elasticsearch.endpoints[x].scope` is `External`)

This is a sample output:

```
status:
  components:
    elasticsearch:
      endpoints:
      - authentication:
          secret:
            secretName: icp4ba-es-auth
          type: BasicSecret
        caSecret:
          key: ca.crt
          secretName: iaf-system-elasticsearch-es-ss-cacert-kp
        name: iaf-system-es
        scope: External
        type: API
        uri: https://iaf-system-es-demo-project.apps.bottled.cp.fyre.ibm.com
      - authentication:
          secret:
            secretName: icp4ba-es-auth
          type: BasicSecret
        caSecret:
          key: ca.crt
          secretName: iaf-system-elasticsearch-es-ss-cacert-kp
        name: iaf-system-elasticsearch-es
        scope: Internal
        type: API
        uri: https://iaf-system-elasticsearch-es.demo-project:9200
```

---

**Parent topic:** [Defining a federated data repository for Process Federation Server containers](./Defining-a-federated-data-repository.md)

**Index:** [Documentation index](../README.md#documentation-index)
