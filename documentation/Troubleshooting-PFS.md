# Troubleshooting Process Federation Server in a container environment

If you face any issue with Process Federation Server, you can reach IBM® support which will ask you to set up relevant logs and gather these logs and information about your configuration.

## Customizing logging

By default, logs are configured for an integration with [Openshift Cluster Logging](https://www.ibm.com/docs/en/openshift?source=https%3A%2F%2Fdocs.openshift.com%2Fcontainer-platform%2F4.3%2Flogging%2Fcluster-logging.html&referrer=SS8JB4_20.x%2Fcom.ibm.wbpm.main.doc%2Ftopics%2Fcon_pfs_tbshoot.html). For this purpose, all logs are output to stdout in JSON format, and are also stored in a file named `/logs/application/liberty-message.log` in basic format by default.

You can customize logging using [pfs_configuration.logs.*](https://www.ibm.com/docs/en/cloud-paks/cp-biz-automation/23.0.2?topic=reference-cp4ba-process-federation-server-parameters) CR values as following:
```
pfs_configuration:
   logs:
       console_format: "json"
       console_log_level: "INFO"
       console_source: "message,trace,accessLog,ffdc,audit"
       trace_format: "ENHANCED"
       trace_specification: "*=info"
```

You can also customize logging by providing your own [configuration dropin](https://www.ibm.com/docs/en/was-liberty/core?topic=files-using-configuration-dropins-folder-specify-server-configuration) through the [pfs_configuration.config_dropins_overrides_secret](https://www.ibm.com/docs/en/cloud-paks/cp-biz-automation/23.0.2?topic=reference-cp4ba-process-federation-server-parameters) CR value.

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
To see the logs, run the oc logs PFS_pod_name command to see the logs, or log into Process Federation Server.
The following example shows how to check the Process Federation Server container logs:
```
$ oc exec -it icp4adeploy-pfs-0 bash
$ cat /logs/application/icp4adeploy-pfs-0/liberty-message.log
```

The above configuration:
* outputs message logs in basic format in `/logs/application/message.log` file
* outputs traces in `/logs/application/trace.log` file. You can adjust the `traceSpecification` property to get the desired detailed traces. IBM support can indicate which relevant trace specification is needed to investigate your issue.

> **Note:** If your pod was already created with [pfs_configuration.config_dropins_overrides_secret](https://www.ibm.com/docs/en/cloud-paks/cp-biz-automation/23.0.2?topic=reference-cp4ba-process-federation-server-parameters) referencing a [secret](https://kubernetes.io/fr/docs/concepts/configuration/secret/), you can dynamically update the content of this secret, and the configuration will be updated without requiring to restart the pod.

## Collecting logs and other data to diagnose issues

The procedure is detailed in IBM Support technote [MustGather: Collecting data to diagnose issues with IBM Process Federation Server in a container](https://www.ibm.com/support/pages/mustgather-collecting-data-diagnose-issues-ibm-process-federation-server-container)

--- 

**Parent topic:** [Maintaining, monitoring and troubleshooting IBM Process Federation Server in a container environment](./Maintaining-monitoring-and-troubleshooting.md)

**Index:** [Documentation index](../README.md#documentation-index)
