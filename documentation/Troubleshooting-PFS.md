# Troubleshooting Process Federation Server in a container environment

If you face any issue with Process Federation Server, you can reach IBM® support which will ask you to set up relevant logs and gather these logs and information about your configuration.

## Customizing logging

By default, logs are configured for an integration with [Openshift Cluster Logging](https://www.ibm.com/docs/en/openshift?source=https%3A%2F%2Fdocs.openshift.com%2Fcontainer-platform%2F4.3%2Flogging%2Fcluster-logging.html&referrer=SS8JB4_20.x%2Fcom.ibm.wbpm.main.doc%2Ftopics%2Fcon_pfs_tbshoot.html). For this purpose, all logs are output to stdout in JSON format, and are also stored in a file named `/logs/application/liberty-message.log` in basic format by default.

You can customize logging using [pfs_configuration.logs.*](https://www.ibm.com/docs/en/baw/20.x?topic=workflow-business-automation-server-parameters#ref_baw_params__pfs) CR values.

You can also customize logging by providing your own [configuration dropin](https://www.ibm.com/docs/en/was-liberty/core?topic=files-using-configuration-dropins-folder-specify-server-configuration) through the [pfs_configuration.config_dropins_overrides_secret](https://www.ibm.com/docs/en/baw/20.x?topic=workflow-business-automation-server-parameters#ref_baw_params__pfs) CR value.

```
<server description="IBM Process Federation Server">
    <logging
        messageFileName="message.log"
        messageFormat="basic"
        messageSource="message,ffdc,"
        consoleFormat="basic"
        consoleLogLevel="INFO"
        consoleSource="message,ffdc"
        traceFormat="ENHANCED"
        traceSpecification="*=info" />
</server>
```

The above configuration:
* outputs message logs in basic format in `/logs/application/message.log` file
* outputs traces in `/logs/application/trace.log` file. You can adjust the `traceSpecification` property to get the desired detailed traces. IBM support can indicate which relevant trace specification is needed to investigate your issue.

> **Note:** If your pod was already created with [pfs_configuration.config_dropins_overrides_secret](https://www.ibm.com/docs/en/baw/20.x?topic=workflow-business-automation-server-parameters#ref_baw_params__pfs) referencing a [secret](https://kubernetes.io/fr/docs/concepts/configuration/secret/), you can dynamically update the content of this secret, and the configuration will be updated without requiring to restart the pod.

## Gathering logs

If the Process Federation Server pods are running, you can execute the following commands to retrieve the configuration and the logs (replace _<pfs_pod_name>_ with the actual pod name):

* Linux/Unix:
  ```
  kubectl exec -it <pfs_pod_name> -- bash -c 'tar -czvf /tmp/mustgather.tar.gz /logs/application/ $(find /config/ -name "*.xml")'
  kubectl cp <pfs_pod_name>:/tmp/mustgather.tar.gz ./mustgather.tar.gz
  kubectl exec -it <pfs_pod_name> -- rm /tmp/mustgather.tar.gz
  ```

* Windows:
  ```
  kubectl exec -it <pfs_pod_name> -- bash -c "tar -czvf /tmp/mustgather.tar.gz /logs/application/ $(find /config/ -name '*.xml')" 
  kubectl cp <pfs_pod_name>:/tmp/mustgather.tar.gz mustgather 
  kubectl exec -it <pfs_pod_name> -- rm /tmp/mustgather.tar.gz
  ```
  
If the Process Federation Server pods are in `CrashLoopBackOff` state, to collect logs, you must access the NFS server and zip up the folder mounted by the persistent volume storing the logs (see section [Persistent volumes](./PFS-Statefulset.md#persistent-volumes) for more details). To identify the NFS server and folder internally used by the pod, you can execute the following command:

* Linux/Unix:
  ```
  kubectl describe pv $(kubectl get pvc logs-<pfs_pod_name> -o=jsonpath='{.spec.volumeName}')
  ```
  
* Windows:
  ```
  kubectl get pvc logs-<pfs_pod_name> -o=jsonpath={.spec.volumeName} > pvc.tmp && set /P PFS_PVC=< pvc.tmp && kubectl describe pv %PFS_PVC% && del pvc.tmp
  ```

If the Process Federation Server pods are in `CrashLoopBackOff` state, to provide information about the configuration:

* send the output of:
  ```
  kubectl describe pod <pfs_pod_name>
  ```
* send the content of the secret referenced as [pfs_configuration.config_dropins_overrides_secret](https://www.ibm.com/docs/en/baw/20.x?topic=workflow-business-automation-server-parameters#ref_baw_params__pfs) in the ICP4ACluster resource specification

If no Process Federation Server pod has been created, send the output of the following command:
```
kubectl describe statefulset <pfs_statefulset_name>
```

You should also send us the output of the following command:
```
kubectl get pods
```

And for any pod which shows an error status, send the output of the following commands:
```
kubectl describe pod <pod_name>
kubectl logs pod <pod_name> -c <container_name>
```
Where *<container_name>* should be replaced by the names of the containers and init-containers as found in the output of:
```
kubectl describe pod <pod_name>
```

You should also:

* send the output of:
  ```
  kubectl get statefulset
  ```

* for any [statefulset](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/) where CURRENT number of replicas is not equal to the DESIRED number of replicas, send the output of:
  ```
  kubectl describe statefulset <statefulset_name>
  ```

* send the output of:
  ```
  kubectl get deployment
  ```

* for any [deployment](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/) where CURRENT number of replicas is not equal to the DESIRED number of replicas, send the output of:
  ```
  kubectl describe deployment <deployment_name>
  ```

> **Note:** If you are using Openshift, you can replace kubectl with oc.

--- 

**Parent topic:** [Maintaining, monitoring and troubleshooting IBM Process Federation Server in a container environment](./Maintaining-monitoring-and-troubleshooting.md)

**Index:** [Documentation index](../README.md#documentation-index)
