# IBM Process Federation Server statefulset

IBM Process Federation Server is deployed as a [statefulset](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/) named `<cr-instance-name>-pfs`.

Each pod of the [statefulset](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/) runs:
* a container (`pfs`) that runs IBM Process Federation Server server
* an init container (`tls-init`) that creates the truststore and keystore used by IBM Process Federation Server container.
* an init container (`folder-prepare-pfs-init`) that prepares the IBM Process Federation Server container filesystem for readonly operations

## Configuration

A default IBM Process Federation Server configuration is provided as part of the docker image in `/config/server.xml` and also mounted from a [secret](https://kubernetes.io/fr/docs/concepts/configuration/secret/)
(`<cr-instance-name>-pfs-config-secret`) in the `/config/configDropins/defaults` directory.

You can override this default configuration by creating a [secret](https://kubernetes.io/fr/docs/concepts/configuration/secret/)
that contains XML [configuration dropins](https://www.ibm.com/support/knowledgecenter/SSD28V_liberty/com.ibm.websphere.wlp.core.doc/ae/twlp_setup_dropins.html),
and reference it as the `config_dropins_overrides_secret` custom resource (CR) value. If you provide this [secret](https://kubernetes.io/fr/docs/concepts/configuration/secret/),
it will be mounted in the 
`/config/configDropins/overrides` directory, enabling you to extend or override the default configuration.

> Note: The Liberty processes the configuration dropins in alphabetical order and you must ensure that the file name of the configuration dropin is alphabetically greater than the file name of the default configuration dropin to override it.

## Accessing the REST API

The base URL to use to connect externally to the IBM Process Federation Server REST API is found in the `status.endpoints` section of the `ProcessFederationServer` custom resource managed by the IBM Process Federation Server operator. This section contains two entries:
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
            "name": "IBM Process Federation Server Internal base URL",
            "scope": "Internal",
            "type": "Service",
            "uri": "https://demo-pfs-service:9443/pfs"
        },
        {
            "name": "IBM Process Federation Server External base URL",
            "scope": "External",
            "type": "Route",
            "uri": "https://my-cp4ba-deployment.mycompany.com/pfs"
        }
    ]
]
```

If the external base URL is _https://my-cp4ba-deployment.mycompany.com/pfs_, the URL to use to call the Swagger UI interface for IBM Process Federation Server REST API is _https://my-cp4ba-deployment.mycompany.com/pfs/rest/bpm/federated/openapi_


## Federation of systems

IBM Process Federation Server running on containers can federate:
- [IBM Business Automation Workflow runtime servers running in the same namespace as IBM Process Federation Server](https://www.ibm.com/docs/baw/25.0.0?topic=containers-federating-business-automation-workflow),
- [IBM Workflow Process Service runtime servers running in the same namespace as IBM Process Federation Server](https://www.ibm.com/docs/cloud-paks/cp-biz-automation/25.0.0?topic=deployment-customizing-workflow-process-service-runtime#tsk_customize_wfps__pfs),
- [IBM Automation Workstream Services servers running in the same namespace as IBM Process Federation Server](https://www.ibm.com/docs/baw/25.0.0?topic=containers-federating-business-automation-workflow),
- [Traditional IBM Business Automation Workflow servers running on-premises](./Federating-traditional-BAW.md).


When federation is enabled for container based runtimes, the CP4BA operator automatically creates a `FederatedSystem` custom resource for each IBM Business Automation Workflow system (BPD and Case) that is deployed in the same namespace as the IBM Process Federation Server containers. The IBM Process Federation Server operator monitors `FederatedSystem` custom resources and creates or removes the [configuration dropin](https://www.ibm.com/support/knowledgecenter/SSD28V_liberty/com.ibm.websphere.wlp.core.doc/ae/twlp_setup_dropins.html) related to these IBM Business Automation Workflow and IBM Cloud Pak for Business Automation servers. These [configuration dropins](https://www.ibm.com/support/knowledgecenter/SSD28V_liberty/com.ibm.websphere.wlp.core.doc/ae/twlp_setup_dropins.html) can be found on the container running IBM Process Federation Server in the `/config/configDropins/overrides` directory. For FederatedSystem parameters, see [Federated system parameters](https://www.ibm.com/docs/cloud-paks/cp-biz-automation/25.0.0?topic=deployment-federated-system-parameters)


## User management

IBM Process Federation Server pods are configured to use the Platform UI services for authentication and single sign-on.

For more information about how authorizations for IBM Process Federation Server users are set up, see [Specifying IBM Process Federation Server user authorizations in the IBM Cloud Pak for Business Automation](./Authorizations.md).

## Persistent volumes

By default, each IBM Process Federation Server pod persists the Liberty server logs folder in a dedicated persistent volume. You can choose to use manual or dynamic provisioning, or to disable the logs persistence, or to change the size of persistent volume, by adapting the [logs.*](https://www.ibm.com/docs/cloud-paks/cp-biz-automation/25.0.0?topic=reference-cp4ba-process-federation-server-parameters) CR values.

---

**Parent topic:** [Architecture of an IBM Process Federation Server container environment](./Architecture.md)

**Index:** [Documentation index](../README.md#documentation-index)
