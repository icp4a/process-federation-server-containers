# Using Elasticsearch when running Process Federation Server with IBM Cloud Pak for Business Automation

Process Federation Server uses a [remote Elasticsearch service](https://www.ibm.com/support/knowledgecenter/en/SS8JB4/com.ibm.wbpm.main.doc/topics/cfg_fps_elasticsearch_remote.html) to store data.

Depending if you are installing a stand-alone IBM® Business Automation Workflow on containers or an IBM Business Automation Workflow Server (or Workflow Runtime) in the IBM Cloud Pak for Business Automation, providing Elasticsearch to Process Federation Server differs.

If you are installing a __stand-alone IBM Business Automation Workflow on containers__, the ways to provide Elasticsearch to Process Federation Server are:

* __standalone-BAW-ES:__ let stand-alone IBM Business Automation Workflow on containers create a statefulset which runs Elasticsearch. This is the default option on Intel architectures (AMD64 or x86_64 the 64-bit edition for Linux x86). This option is not supported on IBM Z (s390x) architectures.

* __External-ES:__ provide your own Elasticsearch separately and reference it in the ICP4ACluster Custom Resource

If you are installing __IBM Cloud Pak for Business Automation__, the ways to provide Elasticsearch to Process Federation Server are:

* __IAF-ES:__ let the IBM Cloud Pak for Business Automation provision and use Elasticsearch provided by IBM® Automation Foundation (IAF). This is the default option.

* __External-ES:__ provide your own Elasticsearch separately and reference it in the ICP4ACluster Custom Resource. This option is only available if `spec.shared_configuration.sc_deployment_type` is set to "Production" (V21.0.3)  or "enterprise"  (V21.0.2 and earlier).

> Note: The deployment type is determined by the spec.shared_configuration.sc_deployment_type property of the ICP4ACluster Custom Resource which can be set to either:
> * Starter or Production (V21.0.3)
> * demo or enterprise (V21.0.2 and earlier)

The following table summarizes the options when providing Elasticsearch in Process Federation Server:

| Delivery | Deployment type | External-ES | IAF-ES | standalone-BAW-ES |
| :---     | :---            | :---        | :---   | :---              |
| CP4BA    | Starter/demo    | Not supported | Supported | Not supported |
| CP4BA    | Production/enterprise    | Supported | Supported | Not supported |
| BAW      | Production/enterprise    | Supported | Not supported | Supported for AMD64 or x86_64 only |

* **[Using Elasticsearch provided by IBM Automation Foundation](./Using-IAF-Elasticsearch.md)**

  When deploying the IBM Cloud Pak for Business Automation, by default, the ICP4ACluster Custom Resource is configured to provision and use Elasticsearch provided by IBM Automation Foundation.
  
* **[Deploying Elasticsearch as part of the stand-alone IBM Business Automation Workflow on containers deployment](./Using-standalone-BAW-Elasticsearch.md)**

  When deploying stand-alone IBM Business Automation Workflow on containers on Intel architectures (AMD64 or x86_64 the 64-bit edition for Linux x86), by default the ICP4ACluster Custom Resource is configured to create a [statefulset](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/), `<icp4acluster-instance-name>-elasticsearch-statefulset`, that deploys pods running Elasticsearch. This option is not supported on IBM Z (s390x) architectures.

* **[Referencing your own Elasticsearch](./Using-own-Elasticsearch.md)**

  Instead of using Elasticsearch as part of the stand-alone IBM Business Automation Workflow on containers deployment or the IBM Business Automation Workflow Server (or Workflow Runtime) in the IBM Cloud Pak for Business Automation deployment, you can decide to reference your own Elasticsearch.

---

**Parent topic:** [Architecture of a Process Federation Server container environment](./Architecture.md)

**Index:** [Documentation index](../README.md#documentation-index)
