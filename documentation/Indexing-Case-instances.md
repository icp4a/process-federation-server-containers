# Indexing case instances

The case management tools provide support for indexing case instances in a federated data repository index. Full reindexing and live index updates are supported.

## About this task

After enabling case federation, the federated data repository index rebuild will need to be triggered by an administrator or operator for the historical case instances to be indexed. The first indexing must be performed before a case management system is registered to Process Federation Server. Subsequent full reindexing is rarely needed, but you can initiate full reindexing if you find that the index is not up to date. Case instances with the following properties are indexed:

* Fixed well-known metadata properties
* Business properties (case properties) defined in the Search view or Summary view of the case solution with the exception of Business objects.

## Procedure

To index case instances:

1. Using a browser extension or similar tool, start the REST API by specifying the following syntax and URL (where *TOS_name* is the symbolic name of the target object store):
   ```
   curl -X POST -u P8Admin https://server:port/CaseManager/CASEREST/v1/TOS_name/index
   ```

1. If you want to perform a partial reindex, use the syntax and URL above but also specify the following parameter (where *UTC_time* is the time that is specified in UTC):
   ```
   {"skipCaseInstancesNotModifiedSince" : "UTC_time"}
   ```
   For example,
   ```
   {"skipCaseInstancesNotModifiedSince" : "2020-09-05 12:00:00"}
   ```
   Partial reindexing is automatically performed in certain situations, such as when you add a new case property in case summary or case search views and then redeploy. In this particular situation, all of the case instances under the case type are reindexed to capture the values of the new fields for the case instances. However, there are situations where you can want to manually perform partial re-indexing by using the `skipCaseInstancesNotModifiedSince` parameter, such as when index inconsistencies are suspected after a disaster recovery.
   
--- 

**Parent topic:** [Administering and operating IBM Process Federation Server](../README.md)

**Index:** [Documentation index](../README.md#documentation-index)
