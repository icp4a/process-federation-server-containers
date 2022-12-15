# Rebuilding federated systems indexes

If you want to delete an Elasticsearch index containing data from a federated system, you can use the following procedure, that does not require you to restart any pod.

## Procedure

1. Open a terminal on a container running within a Process Federation Server pod from the Console UI of your Openshift cluster or using CLI:
   ```
   kubectl exec -it <icp4acluster-instance-name>-pfs-0 -- bash
   ```

1. List the existing indexes:
   ```
   curl -k -u<Elasticsearch_Username>:<Elasticsearch_Password> $ES_ENDPOINTS/_cat/indices
   ```
   Replace _<Elasticsearch_Username>_ and _<Elasticsearch_Password>_ with the Elasticsearch username and password that you can retrieve as follows:
   * if your topology uses Elasticsearch from IBM Automation Foundation, the username and password are in the secret named `icp4ba-es-auth`
   * for other topologies, the username and password are in the secret referenced as `elasticsearch_configuration.admin_secret_name` in the ICP4ACluster custom resource.

1. Delete the index containing data about a given federated system:
   ```
   curl -X DELETE -k -u$<Elasticsearch_Username>:<Elasticsearch_Password> $ES_ENDPOINTS/<Index_Name>
   ```
   After the deletion of the index, Process Federation Server automatically rebuilds it.
   
> **Note:** If you have a very large volume of data to index, the rebuilding may take a few hours.

--- 

**Parent topic:** [Maintaining, monitoring and troubleshooting IBM Process Federation Server in a container environment](./Maintaining-monitoring-and-troubleshooting.md)

**Index:** [Documentation index](../README.md#documentation-index)
