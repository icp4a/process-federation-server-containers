# Migrating saved searches from Elasticsearch 6 to Elasticsearch 7

In Process Federation Server V20.0.0.2, using an external Elasticsearch 6.x cluster is no longer supported. Instead, an external Elasticsearch 7.x (from 7.8.0) cluster is set up (see Elasticsearch product end of life dates at https://www.elastic.co/support/eol)

> **Note:** This procedure applies only for upgrades to V20.0.0.2. If you migrate to Process Federation Server Containers V21.0.2, follow the procedure described at [Migrating Process Federation Server containers to V21.0.2](https://www.ibm.com/docs/en/cloud-paks/cp-biz-automation/21.0.x?topic=mad-migrating-process-federation-server-containers-from-2101-2102).

The migration to Elasticsearch 7 clusters brings in two major changes:

* There no longer is a document type for the documents stored by Process Federation Server in Elasticsearch indices (as a result of the mapping types removal in Elasticsearch 7: [Removal of mapping types](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/removal-of-types.html))

* As index names starting with _"."_ are deprecated in Elasticsearch 7 (see [Create index API](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/indices-create-index.html)), when a federated system is declared with an index name of _myIndexName_, the created index has the name _@myIndexName_. In Elasticsearch 6, an index named _.myIndexName_ was created with an alias of _myIndexName_.

Because Process Federation Server Containers stores the federated saved searches in an Elasticsearch index, if you migrate to Process Federation Server Containers V20.0.0.2 from an existing topology based on an Elasticsearch 6.x cluster, you must export your existing saved searches before upgrading.

> **Note:** If you need information about how to access the Process Federation Server Containers REST APIs, see [Accessing the REST API](./PFS-Statefulset.md#accessing-the-rest-api)

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
   For more details about the `GET /rest/bpm/federated/v1/searches/transfer` REST API, see [Saved Search Transfer Import Resource - GET Method](https://www.ibm.com/docs/en/baw/20.x?topic=import-get)

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
   For more details about the POST /rest/bpm/federated/v1/searches/transfer REST API, see [Saved Search Transfer Import Resource - POST Method](https://www.ibm.com/docs/en/baw/20.x?topic=import-post).
   
--- 

**Parent topic:** [Administering and operating IBM Process Federation Server](../README.md)

**Index:** [Documentation index](../README.md#documentation-index)

