# Migrating to a new Federated Data Repository

If you already have an existing IBM Process Federation Server deployment which relies on a Federated Data Repository, if you plan to switch to another Federated Data Repository, then migration of the data has to be planned.

In the Federated Data Repository, you will find:
- indexes containing docs related to tasks and process instances for each IBM Business Automation Workflow process federated system (BPD),
- indexes related to case instances for each IBM Business Automation Workflow case federated system,
- an index containing saved searches definitions. Such saved search definitions are used with the `/pfs/rest/bpm/federated/v1` search APIs.
- indexes containing reusable queries (one index associated to IBM Process Federation Server, and one index associated to the BAW system which have FDR indexing ). Such reusable queries are used with the `/pfs/rest/bpm/federated/v2` search APIs.

## Indexes related to federated systems

For indexes related to federated systems:
- the indexes related to IBM Business Automation Workflow process federated systems (BPD) will be automatically rebuilt from scratch.
- the indexes related to IBM Business Automation Workflow case federated systems (Case) will have to be rebuilt by following the [Case indexing procedure](https://www.ibm.com/docs/baw/25.0.0?topic=indexes-indexing-case-instances).

## Migrating saved searches

Because IBM Process Federation Server containers stores the federated saved searches in an index of the Federated Data Repository, if you update an existing IBM Process Federation Server containers deployment to use a new Federated Data Repository, you must export your existing saved searches before the update.

> **Note:** If you need information about how to access the IBM Process Federation Server containers REST APIs, see [Accessing the REST API](./PFS-Statefulset.md#accessing-the-rest-api)

1. The user performing the export of the saved searches, and then the import, must be granted the IBM Process Federation Server adminSavedSearch role (see [Specifying IBM Process Federation Server user authorizations on Kubernetes](./Authorizations.md) for more information about how to grant the IBM Process Federation Server adminSavedSearch role to a user).

1. Use the `GET /pfs/rest/bpm/federated/v1/searches/transfer` REST API to export all the IBM Process Federation Server saved searches as a JSON object, such as the following one:
   ```
   {
    
    "results": [
        {
            "importName": "savedSearch1",
            "savedSearch": {
               [...]
            }
        },
        {
            "importName": "savedSearch2",
            "savedSearch": {
               [...]
            }
        },
        [...]
    ],
    "status": 200
   }
   ```
   For more details about the `GET /pfs/rest/bpm/federated/v1/searches/transfer` REST API, see the [IBM Process Federation Server Federated API reference documentation](https://www.ibm.com/docs/baw/25.0.0?topic=apis-rest-interface-process-federation-server-resources)

1. Once IBM Process Federation Server containers have been upgraded to the latest version, re-import the saved searches, by using the `POST /rest/bpm/federated/v1/searches/transfer` REST API to import the array of saved search returned as the results attribute of the previous export:

   ```
   [
        {
            "importName": "savedSearch1",
            "savedSearch": {
               [...]
            }
        },
        {
            "importName": "savedSearch2",
            "savedSearch": {
               [...]
            }
        },
        [...]
   ]
   ```
   For more details about the POST /rest/bpm/federated/v1/searches/transfer REST API, see the [IBM Process Federation Server Federated API reference documentation](https://www.ibm.com/docs/baw/25.0.0?topic=apis-rest-interface-process-federation-server-resources)


## Migrating reusable queries

IBM Process Federation Server containers stores reusable queries in an index of the Federated Data Repository, if you update an existing IBM Process Federation Server containers deployment to use a new Federated Data Repository, you must export your existing reusable queries before the update and then import them back after the migration.

Similarly as for the saved searches, it is also recommended to migrate the reusable queries:

1. Use the `GET /pfs/rest/bpm/federated/v2/search/queries` REST API to export all the IBM Process Federation Server reusable queries as a JSON object.

1. Use the `POST /pfs/rest/bpm/federated/v2/search/queries` REST API to import the IBM Process Federation Server reusable queries individually.

## Embedded IBM Process Federation Server instances when full text search capability is enabled

When enabling the full text search capability in a [IBM Business Automation Workflow server](https://www.ibm.com/docs/cloud-paks/cp-biz-automation/25.0.0?topic=customizing-enabling-full-text-search) running on containers or a [IBM Workflow Process Service server](https://www.ibm.com/docs/cloud-paks/cp-biz-automation/25.0.0?topic=deployment-customizing-workflow-process-service-runtime#tsk_customize_wfps__pfs__title__1), an IBM Process Federation Server embedded runtime is running as part of the same container as the pod running Workflow, and the embedded IBM Process Federation Server REST API can be accessed too under the sames context root as the Workflow server REST API. ex: If you have deployed an IBM Business Automation Workflow authoring server with full text search capability enabled, then you can access the embedded IBM Process Federation Server REST API under `/bas/rest/bpm/federated` context root. Notice the context root `bas` instead of `pfs`.

You should also take care of migrating the saved searches and reusable queries from the embedded IBM Process Federation Server runtimes if any. 

--- 

**Parent topic:** [Administering and operating IBM Process Federation Server](../README.md)

**Index:** [Documentation index](../README.md#documentation-index)

