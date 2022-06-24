# Rebuilding federated systems indexes

If you want to delete an Elasticsearch index containing data from a federated system, you can use the following procedure, that does not require you to restart any pod.

## Procedure

1. Open a terminal on a container running within a pod from the Console UI of your Openshift cluster or using CLI:
   ```
   kubectl exec -it <pod-name> -- bash
   ```

1. List the existing indexes:
   ```
   curl -k -u$ES_USERNAME:<Elasticsearch_Password> $ES_ENDPOINTS/_cat/indices:
   ```
   Replace _<Elasticsearch_Password>_ with the Elasticsearch password that you can retrieve as follows:
   * if your topology uses Elasticsearch from IBM Automation Foundation, the password is in the secret named `icp4ba-es-auth``
   * if your topology uses Elasticsearch deployed as part of the IBM Cloud Pak for Business Automation deployment, the password is in the secret referenced as `elasticsearch_configuration.admin_secret_name` in the ICP4ACluster custom resource.
   * if your topology uses an Elasticsearch that you have setup on your own, then the password is in the secret referenced as `pfs_configuration.external_elasticsearch.admin_secret_name`.

1. Delete the index containing data about a given federated system:
   ```
   curl -X DELETE -k -u$ES_USERNAME:<Elasticsearch_Password> $ES_ENDPOINTS/.6d04b445-4c13-44a8-80b7-f8975a40522a
   ```
   After the deletion of the index, Process Federation Server automatically rebuilds it.
   
> **Note:** If you have a very large volume of data to index, the rebuilding may take a few hours.

--- 

**Parent topic:** [Maintaining, monitoring and troubleshooting IBM Process Federation Server in a container environment](./Maintaining-monitoring-and-troubleshooting.md)

**Index:** [Documentation index](../README.md#documentation-index)
