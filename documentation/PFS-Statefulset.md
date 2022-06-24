# Process Federation Server statefulset

Process Federation Server is deployed as a [statefulset](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/) named `<icp4acluster-instance-name>-pfs`.

Each pod of the [statefulset](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/) runs:
* an init container (`ssl-init-container`) that creates the truststore and keystore used by Process Federation Server container.
* a container (`pfs-node`) that runs Process Federation Server server

## Configuration

A default Process Federation Server configuration is provided as part of the docker image in `/config/server.xml` and also mounted from a [secret](https://kubernetes.io/fr/docs/concepts/configuration/secret/)
(`<icp4acluster-instance-name>-pfs-config-secret`) in the `/config/configDropins/defaults` directory.

You can override this default configuration by creating a [secret](https://kubernetes.io/fr/docs/concepts/configuration/secret/)
that contains XML [configuration dropins](https://www.ibm.com/support/knowledgecenter/SSD28V_liberty/com.ibm.websphere.wlp.core.doc/ae/twlp_setup_dropins.html),
and reference it as the `pfs_configuration.config_dropins_overrides_secret` Custom Resource (CR) value. If you provide this [secret](https://kubernetes.io/fr/docs/concepts/configuration/secret/),
it will be mounted in the 
`/config/configDropins/overrides` directory, enabling you to extend or override the default configuration.

## Accessing the REST API

The Process Federation Server containers expose their HTTPS port. A ClusterIP [service](https://kubernetes.io/docs/concepts/services-networking/service/)
(`<icp4acluster-instance-name>-pfs-service`) exposes the ports of the pods that belong to the Process Federation Server [statefulset](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/).

Depending on the version of IBM® Cloud Pak for Business Automation that you deployed on OpenShift, the route to use to access this service is the following:

### IBM Business Automation Workflow on containers:

A route (`<cr-name>-pfs-route`) is created to expose the service externally using:
* the hostname that you defined as [pfs_configuration.hostname](https://www.ibm.com/docs/en/baw/20.x?topic=workflow-business-automation-server-parameters#ref_baw_params__pfs) CR value
* the port that you defined as [pfs_configuration.port](https://www.ibm.com/docs/en/baw/20.x?topic=workflow-business-automation-server-parameters#ref_baw_params__pfs) CR value

  Or

* the hostname that you defined as [shared_configuration.sc_deployment_hostname_suffix](https://www.ibm.com/docs/en/baw/20.x?topic=workflow-business-automation-server-parameters#ref_baw_params__shared) with `pfs-` as prefix

### For version 21.0.2 and earlier:

A route (`<release>-pfs-route`) is created to expose the service externally using:
* the hostname that you defined as [pfs_configuration.hostname](https://www.ibm.com/docs/en/baw/20.x?topic=workflow-business-automation-server-parameters#ref_baw_params__pfs) CR value
* the port that you defined as [pfs_configuration.port](https://www.ibm.com/docs/en/baw/20.x?topic=workflow-business-automation-server-parameters#ref_baw_params__pfs) CR value

  Or

* the hostname that you defined as [shared_configuration.sc_deployment_hostname_suffix](https://www.ibm.com/docs/en/baw/20.x?topic=workflow-business-automation-server-parameters#ref_baw_params__shared) with `pfs-` as prefix

### On OpenShift, from version 21.0.3:

The [service](https://kubernetes.io/docs/concepts/services-networking/service/) is exposed externally through
the unique Platform UI entry point of the IBM Cloud Pak for Business Automation deployment:
* the URL for the unique entry point is defined by the route named `cpd`
* the path to PFS from this unique endpoint is `/pfs`

For example, if the unique entry point URL is _https://my-cp4ba-deployment.mycompany.com_, the URL to use to call the REST interface for IBM Process Federation Server
resources - Systems Metadata Resource - GET Method is _https://my-cp4ba-deployment.mycompany.com/pfs/rest/bpm/federated/v1/systems_

## Seamless federation

The servers running IBM Business Automation Workflow and/or IBM Automation Workstream Services within stand-alone IBM Business Automation Workflow on containers deployment or within IBM Cloud Pak for Business Automation are seamlessly federated by Process Federation Server:

* A job creates the `PFS_BPD_*` tables in the database configured by `baw_configuration[x].database.*` CR values (see [IBM Business Automation Workflow on containers parameters](https://www.ibm.com/docs/en/SS8JB4_20.x/com.ibm.wbpm.imuc.container.doc/topics/ref_baw_params.html)). The supported database types are DB2, Oracle, and PostgreSQL.

* A resource registry, where IBM Business Automation Workflow instances are registered and unregistered, is also running. Process Federation Server continuously checks for new or deleted entries in the resource registry, and creates or removes the [configuration dropin](https://www.ibm.com/support/knowledgecenter/SSD28V_liberty/com.ibm.websphere.wlp.core.doc/ae/twlp_setup_dropins.html) related to these IBM Business Automation Workflow and IBM Cloud Pak for Business Automation servers. These [configuration dropins](https://www.ibm.com/support/knowledgecenter/SSD28V_liberty/com.ibm.websphere.wlp.core.doc/ae/twlp_setup_dropins.html) can be found on the container running Process Federation Server in the `/config/configDropins/defaults` directory.

* A deployment (`<icp4acluster-instance-name>-pfs-dbareg`) with one replica also runs continuously to maintain an entry in this resource registry which contains the Process Federation Server REST endpoint front end. UIs deployed by stand-alone IBM Business Automation Workflow on containers and IBM Cloud Pak for Business Automation retrieve this entry to later invoke Process Federation Server REST APIs.

Process Federation Server that runs as part of IBM Cloud Pak for Business Automation can also federate IBM Business Automation Workflow running on-premise. For more information, see what Federating IBM Business Automation Workflow running on-premise .
Process Federation Server that runs as part of IBM Cloud Pak for Business Automation can also federate IBM Business Automation Workflow running on-premise. For more information, [Federating IBM Business Automation Workflow running on-premise](./Federating-on-premises-BAW.md).

## User management

User management depends on your deployment and IBM Cloud Pak for Business Automation version:

* Stand-alone IBM Business Automation Workflow on containers: User Management System is configured and you can associate it with a LDAP server.

* IBM Cloud Pak for Business Automation version 21.0.2 and earlier: User Management System is configured and you can associate it with a LDAP server.

* IBM Cloud Pak for Business Automation version 21.0.3: Process Federation Server pods are configured to use the Platform UI services for authentication and single sign-on.

For more information about how authorizations for Process Federation Server users are set up, see [Specifying Process Federation Server user authorizations in the IBM Cloud Pak for Business Automation](./Authorizations.md).

## Persistent volumes

By default, each Process Federation Server pod persists the Liberty server logs folder in a dedicated persistent volume. You can choose to use manual or dynamic provisioning, or to disable the logs persistence, or to change the size of persistent volume, by adapting the [pfs_configuration.logs.*](https://www.ibm.com/docs/en/baw/20.x?topic=workflow-business-automation-server-parameters#ref_baw_params__pfs) CR values.

---

**Parent topic:** [Architecture of a Process Federation Server container environment](./Architecture.md)

**Index:** [Documentation index](../README.md#documentation-index)
