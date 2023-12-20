# Migrating saved searches to a new federated data repository

Because Process Federation Server Containers stores the federated saved searches in an index of the federated data repository, if you update an existing Process Federation Server Containers deployment to use a new federated data repository, you must export your existing saved searches before the update.

> **Note:** If you need information about how to access the Process Federation Server Containers REST APIs, see [Accessing the REST API](./PFS-Statefulset.md#accessing-the-rest-api)

1.  The user performing the export of the saved searches, and then the import, must be granted the PFS adminSavedSearch role (see [Specifying Process Federation Server user authorizations on Kubernetes](./Authorizations.md) for more information about how to grant the PFS adminSavedSearch role to a user).

1. Use the `GET /rest/bpm/federated/v1/searches/transfer` REST API to export all the Process Federation Server saved searches as a JSON object, such as the following one:
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
   For more details about the `GET /rest/bpm/federated/v1/searches/transfer` REST API, see the [IBM Process Federation Server Federated API reference documentation](https://www.ibm.com/docs/en/baw/23.x?topic=apis-rest-interface-process-federation-server-resources)

1. Once Process Federation Server Containers have been upgraded to the latest version, re-import the saved searches, by using the `POST /rest/bpm/federated/v1/searches/transfer` REST API to import the array of saved search returned as the results attribute of the previous export:

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
   For more details about the POST /rest/bpm/federated/v1/searches/transfer REST API, see the [IBM Process Federation Server Federated API reference documentation](https://www.ibm.com/docs/en/baw/23.x?topic=apis-rest-interface-process-federation-server-resources)
   
--- 

**Parent topic:** [Administering and operating IBM Process Federation Server](../README.md)

**Index:** [Documentation index](../README.md#documentation-index)

