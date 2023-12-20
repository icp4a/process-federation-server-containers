# Process Federation Server statefulset

Process Federation Server is deployed as a [statefulset](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/) named `<cr-instance-name>-pfs`.

Each pod of the [statefulset](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/) runs:
* a container (`pfs`) that runs Process Federation Server server
* an init container (`tls-init`) that creates the truststore and keystore used by Process Federation Server container.
* an init container (`folder-prepare-pfs-init`) that prepares the Process Federation Server container filesystem for readonly operations

## Configuration

A default Process Federation Server configuration is provided as part of the docker image in `/config/server.xml` and also mounted from a [secret](https://kubernetes.io/fr/docs/concepts/configuration/secret/)
(`<cr-instance-name>-pfs-config-secret`) in the `/config/configDropins/defaults` directory.

You can override this default configuration by creating a [secret](https://kubernetes.io/fr/docs/concepts/configuration/secret/)
that contains XML [configuration dropins](https://www.ibm.com/support/knowledgecenter/SSD28V_liberty/com.ibm.websphere.wlp.core.doc/ae/twlp_setup_dropins.html),
and reference it as the `pfs_configuration.config_dropins_overrides_secret` Custom Resource (CR) value. If you provide this [secret](https://kubernetes.io/fr/docs/concepts/configuration/secret/),
it will be mounted in the 
`/config/configDropins/overrides` directory, enabling you to extend or override the default configuration.

## Accessing the REST API

The Process Federation Server containers expose their HTTPS ports. A ClusterIP [service](https://kubernetes.io/docs/concepts/services-networking/service/)
(`<cr-instance-name>-pfs-service`) exposes the ports of the pods that belong to the Process Federation Server [statefulset](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/).

The base URL to use to connect externaly to the Process Federation Server REST API is found in the `status.endpoints` section of the `ProcessFederationServer` Custom Resource managed by the Process Federation Server operator. This section contains two entries:
1. One for the service endpoint, which scope is `Internal`
1. One for the external endpoint, which scope is `External`

Those value can be quickly retrieved using the command line (example here using the `oc` client for an Openshift deployment). To list the existing `ProcessFederationServer` custom resources in a namespace, use:

```
oc -n <namespace> get processfederationserver
```

To get the `status.endpoints` section of a specific `ProcessFederationServer` custom resource, use the following command:

```
oc -n <namespace> get processfederationserver <cr-name> -o=jsonpath-as-json='{@.status.endpoints}'
```

Here is a sample result for this last command:

```
[
    [
        {
            "name": "Process Federation Server Internal base URL",
            "scope": "Internal",
            "type": "Service",
            "uri": "https://demo-pfs-service:9443/pfs"
        },
        {
            "name": "Process Federation Server External base URL",
            "scope": "External",
            "type": "Route",
            "uri": "https://my-cp4ba-deployment.mycompany.com/pfs"
        }
    ]
]
```

If the external base URL is _https://my-cp4ba-deployment.mycompany.com_, the URL to use to call the Swagger UI interface for IBM Process Federation Server REST API is _https://my-cp4ba-deployment.mycompany.com/pfs/rest/bpm/federated/openapi_


## Seamless federation

The servers running IBM Business Automation Workflow and/or IBM Automation Workstream Services within stand-alone IBM Business Automation Workflow on containers deployment or within IBM Cloud Pak for Business Automation are seamlessly federated by Process Federation Server:

* A job creates the `PFS_BPD_*` tables in the database configured by `baw_configuration[x].database.*` CR values (see [Workflow Server configuration parameters](https://www.ibm.com/docs/en/cloud-paks/cp-biz-automation/23.0.2?topic=parameters-business-automation-workflow-runtime-workstream-services#ref_baw_params__baw-config)). The supported database types are DB2, Oracle, SQLServer and PostgreSQL.

* When federation is enabled (see [Federating IBM Business Automation Workflow on containers](https://www.ibm.com/docs/en/cloud-paks/cp-biz-automation/23.0.2?topic=deployment-federating-business-automation-workflow-containers)), the CP4BA operator automatically creates a FederatedSystem custom resource for each Business Automation Workflow system (BPD and Case) that is deployed in the same namespace as the Process Federation Server containers. To enable federation, see [Federating IBM Business Automation Workflow on containers](https://www.ibm.com/docs/en/cloud-paks/cp-biz-automation/23.0.2?topic=deployment-federating-business-automation-workflow-containers). The Process Federation Server operator monitors FederatedSystem custom resources and creates or removes the [configuration dropin](https://www.ibm.com/support/knowledgecenter/SSD28V_liberty/com.ibm.websphere.wlp.core.doc/ae/twlp_setup_dropins.html) related to these IBM Business Automation Workflow and IBM Cloud Pak for Business Automation servers. These [configuration dropins](https://www.ibm.com/support/knowledgecenter/SSD28V_liberty/com.ibm.websphere.wlp.core.doc/ae/twlp_setup_dropins.html) can be found on the container running Process Federation Server in the `/config/configDropins/overrides` directory. For FederatedSystem parameters, see [Federated system parameters](https://www.ibm.com/docs/en/cloud-paks/cp-biz-automation/23.0.2?topic=deployment-federated-system-parameters)

Process Federation Server that runs as part of IBM Cloud Pak for Business Automation can also federate IBM Business Automation Workflow running on-premise. For more information, [Federating IBM Business Automation Workflow running on-premise](./Federating-on-premises-BAW.md).

## User management

User management depends on your deployment and IBM Cloud Pak for Business Automation version:

* Stand-alone IBM Business Automation Workflow on containers: User Management System is configured and you can associate it with a LDAP server.

* IBM Cloud Pak for Business Automation: Process Federation Server pods are configured to use the Platform UI services for authentication and single sign-on.

For more information about how authorizations for Process Federation Server users are set up, see [Specifying Process Federation Server user authorizations in the IBM Cloud Pak for Business Automation](./Authorizations.md).

## Persistent volumes

By default, each Process Federation Server pod persists the Liberty server logs folder in a dedicated persistent volume. You can choose to use manual or dynamic provisioning, or to disable the logs persistence, or to change the size of persistent volume, by adapting the [pfs_configuration.logs.*](https://www.ibm.com/docs/en/cloud-paks/cp-biz-automation/23.0.2?topic=reference-cp4ba-process-federation-server-parameters) CR values.

---

**Parent topic:** [Architecture of a Process Federation Server container environment](./Architecture.md)

**Index:** [Documentation index](../README.md#documentation-index)
