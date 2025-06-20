# Using Opensearch provided by IBM Cloud Pak foundational services

When deploying the IBM® Cloud Pak for Business Automation or Business Automation Workflow on Container, Opensearch provided by IBM Cloud Pak foundational services will be installed depending on the selected capability, pattern, and optional components.

To ensure that Opensearch provided by IBM Cloud Pak foundational services is installed, select `Data Collector and Data Indexer` or `Exposed OpenSearch` optional component (these 2 settings will respectively add `pfs` or `opensearch` to the `shared_configuration.sc_optional_components` parameter of the resulting `ICP4ACluster` custom resource).

IBM Cloud Pak foundational services will deploy a opensearch Cluster with name `opensearch` and NodePool object named `opensearch-all` with 3 replicas.

## Accessing the Opensearch cluster REST API

To access the Opensearch REST API, you must retrieve its credentials and endpoint.

### 1. Determine the user secret name 

To retrieve the user secret name query the opensearch cluster as follows:
  
```
kubectl get cluster opensearch -o jsonpath={.spec.plugins.security.internalUserSecret} && echo
```
This command will return the secret name defining the credentials, for example:

```
opensearch-admin-user
```

### 2. Determine the opensearch user 

You can now query the user secrect as follows:

```
kubectl get secret <secret-name> -o jsonpath='{.data}'
```
Replace `<secret-name>` is the name retrieved in the first step. The result looks like:
```
{"opensearch-admin":"<some base 64 string>"}
```
In this case the admin user is `opensearch-admin`.

### 3. Determine the opensearch password

Once you have the user, decode the user's password using the following command:

```
kubectl get secret <secret-name> -o jsonpath='{.data.<opensearch-admin-user>}'  | base64 --decode
```  

Replace `<secret-name>` with the name retrieved in the first step and `<opensearch-admin-user>` with the user name in step 2.


### 4. Determine the opensearch endpoint

When you are on OCP or ROKS, query the opensearch route as follows:

```
oc get route | grep opensearch
```

The second field is the hostname to use to access the Opensearch REST API. 

For other CNCF platforms, query the opensearch ingress as follows:

```
kubectl get ingress opensearch-ingress
```

The value listed as `HOSTS` property shows the the Opensearch REST endpoint.



You can now access Opensearch using a URL to this hostname and the credentials retrieved previously:

```
https://opensearch-<namespace>.mycncf.com
```

---

**Parent topic:** [Defining a federated data repository for Process Federation Server containers](./Defining-a-federated-data-repository.md)

**Index:** [Documentation index](../README.md#documentation-index)