# Federating IBM Business Automation Workflow running on-premise

You can configure IBM® Process Federation Server containers to federate on-premise IBM Business Automation Workflow or IBM Business Process Manager systems.

---
> **Attention:** If the version of Process Federation Server you are using is running on OpenShift with Platform UI service enabled, follow the procedure described in [this Technote](https://www.ibm.com/support/pages/node/6514469) instead of the one documented here.
> 
---

The configuration of each on-premise system requires the following steps:
* [Preparing the on-premises IBM Business Automation Workflow system for federation](#preparing-the-on-premises-ibm-business-automation-workflow).
* [Providing self-signed certificates of the on-premise topology for the Process Federation Server truststore to include them](#providing-self-signed-certificates-of-the-on-premises-topology-so-the-process-federation-server-truststore-can-include-them).
* [Providing the Process Federation Server config-dropin which will declare the on-premises IBM Business Automation Workflow system to federate](#providing-the-process-federation-server-config-dropin-which-will-declare-the-on-premises-ibm-business-automation-workflow-system-to-federate).
* [Configuring Process Portal to work on tasks and processes from the on-premises IBM Business Automation Workflow](#configuration-related-to-process-portal).
* [Configuring Workplace to work on tasks and processes from the on-premises IBM Business Automation Workflow](#configuration-related-to-workplace).

## Preparing the on-premises IBM Business Automation Workflow

#### Enabling indexing on the on-premises IBM Business Automation Workflow system

The distributed Process Federation Server index enables process participants to see a consolidated list of BPD-related tasks from all IBM Business Automation Workflow systems in the federated environment. For data from a federated IBM Business Automation Workflow system to be included in the index, you must enable indexing on the IBM Business Automation Workflow system, as described in [Enabling indexing on a federated system](https://www.ibm.com/docs/en/baw/22.x?topic=systems-enabling-indexing-federated-system).

As part of this configuration, you have to create the change log tables in the Process Server database of the IBM Business Automation Workflow system. In the pod that runs Process Federation Server, the database scripts are located in `/opt/ibm/wlp/ibmProcessFederationServer/wlp-ext/dbscripts/`.

#### Configuring the on-premises IBM Business Automation Workflow system to use the same IBM® User Management Service (UMS) as Process Federation Server

The IBM Business Automation Workflow system must be configured to use the same IBM® UMS as Process Federation Server for Single Sign-On (SSO) as documented in [Adding IBM Business Automation Workflow and IBM Federated Process Portal to use the User Management Service](https://www.ibm.com/docs/en/baw/22.x?topic=css-adding-business-automation-workflow-federated-process-portal-use-user-management-service).

> **Note:** Only IBM Business Automation Workflow and IBM Business Process Manager systems which provide IBM® UMS support can be federated by Process Federation Server running in Kubernetes.
> 

## Providing self-signed certificates of the on-premises topology so the Process Federation Server truststore can include them

If the on-premises IBM Business Automation Workflow system and/or its database exposes self-signed certificates, these certificates must be added to the Process Federation Server truststore. For each of these certificates, you must create a [secret](https://kubernetes.io/docs/concepts/configuration/secret/) resource with an entry named `tls.crt` which contains the public key of the certificate in PEM format.

Once all the self-signed certificates are packaged into secrets, they must be declared under the [pfs_configuration.tls.tls_trust_list](https://www.ibm.com/docs/en/cloud-paks/cp-biz-automation/22.0.1?topic=parameters-business-automation-workflow-runtime-workstream-services#ref_baw_params__pfserver) CR value.

## Providing the Process Federation Server config-dropin which will declare the on-premises IBM Business Automation Workflow system to federate

Once the on-premises IBM Business Automation Workflow system is properly configured to perform indexing and to communicate with IBM® UMS for user authentication, you must provide the proper XML [config-dropin](https://www.ibm.com/docs/en/was-liberty/core?topic=files-using-configuration-dropins-folder-specify-server-configuration) to Process Federation Server.

In the CR values that are used to install IBM Cloud Pak for Business Automation, you can reference a [secret](https://kubernetes.io/fr/docs/concepts/configuration/secret/) that contains XML [configuration dropins](https://www.ibm.com/docs/en/was-liberty/core?topic=files-using-configuration-dropins-folder-specify-server-configuration) as the [pfs_configuration.config_dropins_overrides_secret](https://www.ibm.com/docs/en/cloud-paks/cp-biz-automation/22.0.1?topic=parameters-business-automation-workflow-runtime-workstream-services#ref_baw_params__pfserver) CR value. The entries from this secret will be mapped as files in the `/config/configDropins/overrides` directory of each pod that runs Process Federation Server.

To federate the on-premises IBM Business Automation Workflow system, a new entry must be created in this secret. This entry contains the following configuration elements:

* the declaration of the `<ibmPfs_federatedSystem>` element
* the datasource of the on-premise IBM Business Automation Workflow system (`<datasource>` element)
* the `<ibmPfs_bpdIndexer>` element
* the `<ibmPfs_bpdRetriever>` element

You can refer to [Configuration properties for federated systems](https://www.ibm.com/docs/en/baw/22.x?topic=systems-configuration-properties-federated) for detailed information about the above configuration elements.

> **Note:**
> Process Federation Server containers has a set of pre defined libraries that you can reference when you declare the jdbcDriver used by your datasource:
> * For a DB2 datasource, use `<jdbcDriver libraryRef="DB2JCC4Lib"/>`
> * For an Oracle datasource, use `<jdbcDriver libraryRef="OracleLib"/>`
> * For a Microsoft SQL Server datasource, use `<jdbcDriver libraryRef="MSJDBCLib"/>`

## Configuration related to Process Portal

For Process Portal to start processes and work on tasks from the on-premises IBM Business Automation Workflow system, you must reference the on-premises IBM Business Automation Workflow URL in the CR values related to the IBM Business Automation Workflow instance which hosts Process Portal:

```
baw_configuration:
    - [...]
      host_federated_portal: true
      federated_portal:
        content_security_policy_additional_origins:
          - 'https://onprem.baw.com:9443'
      [...]
```

## Configuration related to Workplace

For Workplace to start processes and work on tasks from the on-premises IBM Business Automation Workflow system, you must create an entry into the Resource Registry (RR):

1. Locate your RR writer username and password. You can find the writer username and password under the secret referenced in the `resource_registry_configuration.admin_secret_name` CR value (see the detailed description in the Resource Registry table on the [IBM Business Automation Application Engine parameters](https://www.ibm.com/docs/en/cloud-paks/cp-biz-automation/22.0.1?topic=parameters-business-automation-application#ref_ae_params__rr_params) page). The writer username and password are respectively set in this secret as `writeUser` and `writePassword` keys.

1. Open a terminal on one of the RR pod. RR pod name should have the following format `<icp4acluster-instance-name>-dba-rr-<alphanumeric-id>`.

1. Create a RR key using the `etcdctl CLI`. If the on-premises IBM Business Automation Workflow endpoint is `https://onprem.baw.com:9443`:
  * the key will be the system Id. The system Id can be retrieved by invoking the https://onprem.baw.com:9443/rest/bpm/wle/v1/systems API (`<systemID>` XML tag);
  * the value will be the following JSON:
    ```
    {"hostname":"onprem.baw.com", "port": "9443", "protocol": "https", "ctxRoot": "/rest/bpm/wle"}
    ```
    > **Note:** The value is the destructuring of the `restUrlPrefix` property of the `ibmPfs_federatedSystem` configuration element.
  To create this key, execute the following command in the terminal (example given here with a system Id of _feded754-790d-4214-a326-865812d4357a_):
    ```
    etcdctl --cacert="$ETCD_PEER_TRUSTED_CA_FILE" --user="writer:writerpwd" --endpoints="https://<icp4acluster-instance-name>-dba-rr-client:2379" put /dba/appresources/IBM_WORKFLOW/WORKFLOW_SYSTEM/feded754-790d-4214-a326-865812d4357a '{"hostname":"onprem.baw.com", "port": "9443", "protocol": "https", "ctxRoot": "/rest/bpm/wle"}'
    ```
  
 > **Note:** Enabling persistence of the Resource Registry is highly recommended in production to avoid having the key disappearing upon restarts of cluster or pods. For more details, see the description of the `resource_registry_configuration.auto_backup.*` CR values in the Resource Registry table on the [IBM Business Automation Application Engine parameters](https://www.ibm.com/docs/en/cloud-paks/cp-biz-automation/22.0.1?topic=parameters-business-automation-application#ref_ae_params__rr_params) page.
--- 

**Parent topic:** [Administering and operating IBM Process Federation Server](../README.md)

**Index:** [Documentation index](../README.md#documentation-index)
