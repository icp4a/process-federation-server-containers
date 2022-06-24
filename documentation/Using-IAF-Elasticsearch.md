# Using Elasticsearch provided by IBM Automation Foundation

When deploying the IBM® Cloud Pak for Business Automation, by default, the ICP4ACluster Custom Resource is configured to provision and use Elasticsearch provided by IBM Automation Foundation.

For more information, see [IBM Automation Foundation operational data store](https://www.ibm.com/docs/en/cloud-paks/1.0?topic=configuration-operational-datastore).

IBM Automation Foundation will deploy a [statefulset](https://kubernetes.io/fr/docs/concepts/workloads/controllers/statefulset/) named `iaf-system-elasticsearch-es-data` with the following number of replicas:

* 1 replica if `spec.shared_configuration.sc_deployment_type` property of the ICP4ACluster Custom Resource is set to "Starter" or "demo"; or if `spec.shared_configuration.sc_deployment_profile_size` is set to "small" or "medium";

* 3 replicas if `spec.shared_configuration.sc_deployment_type` property of the ICP4ACluster Custom Resource is set to "Production"  or "enterprise".

## Accessing the Elasticsearch cluster REST API

To access the Elasticsearch REST API, you must retrieve its route and credentials by executing the command:

```
oc get cartridgerequirements icp4ba -o yaml
```

Elasticsearch access information is in the returned content:
* `Status.Components.Elasticsearch.Endpoints.Uri` (where scope is `External`)
* `Status.Components.Elasticsearch.Endpoints.Authentication.Secret.SecretName`(where scope is `External`)

This is a sample output:

```
Status:
  Components:
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
        uri: https://iaf-system-es-baw1.apps.bawdev-x301.cp.fyre.ibm.com
      - authentication:
          secret:
            secretName: icp4ba-es-auth
          type: BasicSecret
        caSecret:
          key: ca.crt
          secretName: iaf-system-elasticsearch-es-ss-cacert-kp
        name: iaf-system-elasticsearch-es
        scope: External
        type: API
        uri: https://iaf-system-es-baw1.apps.mydeployment.cp.fyre.ibm.com
```

---

**Parent topic:** [Using Elasticsearch when running Process Federation Server with IBM Cloud Pak for Business Automation](./Using-Elasticsearch.md)

**Index:** [Documentation index](../README.md#documentation-index)
