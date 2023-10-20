# Monitoring IBM Process Federation Server

You can monitor a running server to gather information, detect issues, and perform actions without having to restart the server.

## Accessing the Process Federation Server MBeans

[JMX Mbeans](https://www.ibm.com/docs/en/baw/23.x?topic=server-monitoring-administering-process-federation) are deployed with Process Federation Server to monitor and perform administrative tasks for Process Federation Server servers.

The MBeans execution must be performed on a given pod. As a Kubernetes [service](https://kubernetes.io/docs/concepts/services-networking/service/) is created to expose all the pods of the Process Federation Server [statefulset](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/), if you have more than one replica in your Process Federation Server [statefulset](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/), this [service](https://kubernetes.io/docs/concepts/services-networking/service/) cannot be used to access the MBeans. Instead, you must open a terminal on a Process Federation Server pod, and then access the MBean on `localhost`.

You can open a terminal on a container running within a Process Federation Server pod from the Console UI of your Openshift cluster, or using CLI:

```
kubectl exec -it <icp4acluster-instance-name>-pfs-0 -- bash
```

A pre requisite to access the MBeans is to have at least one user with the `administrator-role` authorization. [Specifying Process Federation Server user authorizations on Kubernetes](./Authorizations.md) provides detailed instructions on how to do that.

You can then access the MBeans using Curl or using the admin scripts which are located in the `/opt/ibm/wlp/ibmProcessFederationServer/wlp-ext/adminScripts` directory on the running pod.

To use the admin scripts, you must also define the following environment variables as a prerequisite:

```
export WLP_HOME=/opt/ibm/wlp/
export PFS_ADMIN_HOME=/opt/ibm/wlp/ibmProcessFederationServer/wlp-ext/adminScripts
export PFS_JAVA_OPTS=-Dcom.ibm.ws.jmx.connector.client.disableURLHostnameVerification=true
```

Here is an example of the options file that will be passed as argument :

```
host=localhost
port=9443
user=<id_of_the_administrator_user>
password=<password_of_the_administrator_user>
trustStoreFile=/opt/ibm/wlp/usr/servers/defaultServer/resources/generated_security/truststore/jks/trusts.jks
trustStoreType=JKS
trustStorePassword=passw0rd
```

You can then execute the scripts, for example:

```
sh-4.2$ ./listBpdIndexerUniqueIDs.sh -file options -verbose
[2020-05-13 12:11:34.758 UTC] CWMF60000I: Establishing connection with service:jmx:rest://localhost:9443/IBMJMXConnectorREST
[2020-05-13 12:11:38.039 UTC] CWMF60001I: Connection established
[2020-05-13 12:11:38.039 UTC] CWMF60005I: Retrieving the list of identifiers for all the active target components 
[2020-05-13 12:11:38.211 UTC] CWMF60006I: Command executed, result: [id=6d04b445-4c13-44a8-80b7-f8975a40522a.default-0]
[2020-05-13 12:11:38.211 UTC] CWMF60010I: Closing connection
```

--- 

**Parent topic:** [Maintaining, monitoring and troubleshooting IBM Process Federation Server in a container environment](./Maintaining-monitoring-and-troubleshooting.md)

**Index:** [Documentation index](../README.md#documentation-index)
