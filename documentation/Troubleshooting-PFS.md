# Troubleshooting IBM Process Federation Server in a container environment

If you face any issue with IBM Process Federation Server, you can reach IBM support which will ask you to set up relevant logs and gather these logs and information about your configuration.

## Customizing logging

By default, logs are configured for an integration with [Openshift Cluster Logging](https://www.ibm.com/docs/en/openshift?source=https%3A%2F%2Fdocs.openshift.com%2Fcontainer-platform%2F4.3%2Flogging%2Fcluster-logging.html&referrer=SS8JB4_20.x%2Fcom.ibm.wbpm.main.doc%2Ftopics%2Fcon_pfs_tbshoot.html). For this purpose, all logs are output to stdout in JSON format, and are also stored in a file named `/logs/application/liberty-message.log` in basic format by default.

You can customize logging using [logs.*](https://www.ibm.com/docs/cloud-paks/cp-biz-automation/25.0.0?topic=reference-cp4ba-process-federation-server-parameters) CR values as following:
```
pfs_configuration:
   logs:
       console_format: "json"
       console_log_level: "INFO"
       console_source: "message,trace,accessLog,ffdc,audit"
       trace_format: "ENHANCED"
       trace_specification: "*=info"
```

You can also customize logging by providing your own [configuration dropin](https://www.ibm.com/docs/en/was-liberty/core?topic=files-using-configuration-dropins-folder-specify-server-configuration) through the [config_dropins_overrides_secret](https://www.ibm.com/docs/cloud-paks/cp-biz-automation/25.0.0?topic=reference-cp4ba-process-federation-server-parameters) CR value.

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
To see the logs, run the `oc logs <IBM Process Federation Server_pod_name>` command to see the logs, or log into IBM Process Federation Server.
The following example shows how to check the IBM Process Federation Server container logs:
```
$ oc exec -it pfsdeploy-pfs-0 bash
$ cat /logs/application/pfsdeploy-pfs-0/liberty-message.log
```

The above configuration:
* outputs message logs in basic format in `/logs/application/pfsdeploy-pfs-0/message.log` file
* outputs traces in `/logs/application/pfsdeploy-pfs-0/trace.log` file. You can adjust the `traceSpecification` property to get the desired detailed traces. IBM support can indicate which relevant trace specification is needed to investigate your issue.

## Collecting logs and other data to diagnose issues

The procedure is detailed in IBM Support technote [MustGather: Collecting data to diagnose issues with IBM Process Federation Server in a container](https://www.ibm.com/support/pages/mustgather-collecting-data-diagnose-issues-ibm-process-federation-server-container)

--- 

**Parent topic:** [Maintaining, monitoring and troubleshooting IBM Process Federation Server in a container environment](./Maintaining-monitoring-and-troubleshooting.md)

**Index:** [Documentation index](../README.md#documentation-index)
