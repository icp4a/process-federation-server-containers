# Administering and operating IBM Process Federation Server Containers

IBM® Process Federation Server is a component for IBM Business Automation Workflow (BAW) and IBM Business Process Manager (BPM) environments. This component creates a federated process environment that provides business users with a single point of access to their task list and launch list, regardless of the type of process that they are working on and the BAW and / or BPM back-end system on which the process artifacts are stored.

When IBM Business Automation Workflow traditional offering is deployed on-premises, IBM Process Federation Server is an optional component that is extensively documented in the [IBM Business Automation Workflow Knowledge Center](https://www.ibm.com/docs/en/baw/20.x?topic=server-traditional-installing-enabling-process-federation).

When IBM Business Automation Workflow containers are deployed on [Red Hat Openshift Container Platform](https://www.redhat.com/en/technologies/cloud-computing/openshift/container-platform) or other [CNCF Kubernetes platforms](https://www.cncf.io/projects/kubernetes/), Process Federation Server containers are also automatically deployed in order to provide a federated process environment that can be configured to federate on-premises Business Automation Workflow systems (V18.0.0.1 or later) with the Business Automation Workflow containers runtimes.

For information about installing Business Automation Workflow on containers, see:
* for stand-alone IBM Business Automation Workflow on containers: [Containers: Installing IBM Business Automation Workflow](https://www.ibm.com/docs/en/SS8JB4_20.x/com.ibm.wbpm.imuc.container.doc/topics/con_baw_inst.html);
* for IBM Business Automation Workflow deployment with IBM Cloud Pak for Business Automation: [Installing IBM Cloud Pak for Business Automation](https://www.ibm.com/docs/en/cloud-paks/cp-biz-automation/21.0.x?topic=automation-installing)

The instructions for stand-alone IBM Business Automation Workflow on containers are similar to the instructions for the Business Automation Workflow Server (or Workflow Runtime) in the IBM Cloud Pak for Business Automation, and they both involve the creation of a Custom Resource of the ICP4ACluster kind. This Custom Resource is referenced as the _ICP4ACluster Custom Resource_ in this Process Federation Server containers documentation.

Since IBM® Cloud Pak for Business Automation (CP4BA) 22.0.1, Process Federation Server can be installed separately without BAW. For information about installing Process Federation Server separately, see [installing Process Federation Server Deployments](https://www.ibm.com/docs/SSYHZ8_22.0.1/com.ibm.dba.install/op_topics/tsk_prepare_pfs.html)


## Documentation index

* **[Architecture of a Process Federation Server container environment](./documentation/Architecture.md)**

  When Process Federation Server is deployed using an ICP4ACluster Custom Resource, several kubernetes resources are created. The topics in this section provide detailed information about those resources.
  
  * **[Process Federation Server statefulset](./documentation/PFS-Statefulset.md)**

    Process Federation Server is deployed as a [statefulset](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/) named `<icp4acluster-instance-name>-pfs`. This section provide detailed information about this statefulset. 

  * **[Using Elasticsearch when running Process Federation Server containers](./documentation/Using-Elasticsearch.md)**

    Process Federation Server uses a [remote Elasticsearch service](https://www.ibm.com/docs/en/baw/20.x?topic=service-configuring-remote-elasticsearch) to store data. The topics in this section provide detailed information about the different types of Elasticsearch cluster that can be used by PFS containers, depending on the type of IBM Business Automation Workflow containers installation:

    * **[Using Elasticsearch provided by IBM Automation Foundation](./documentation/Using-IAF-Elasticsearch.md)**

      When deploying IBM Business Automation Workflow containers with IBM Cloud Pak for Business Automation, by default, the ICP4ACluster Custom Resource is configured to provision and use Elasticsearch provided by IBM Automation Foundation. This section provides information about using this type of Elasticsearch cluster.

    * **[Deploying Elasticsearch as part of the stand-alone IBM Business Automation Workflow on containers deployment](./documentation/Using-standalone-BAW-Elasticsearch.md)**

        When deploying stand-alone IBM Business Automation Workflow on containers on Intel architectures (AMD64 or x86_64 the 64-bit edition for Linux x86), by default the ICP4ACluster Custom Resource is configured to create a statefulset, <icp4acluster-instance-name>-elasticsearch-statefulset, that deploys pods running Elasticsearch. This option is not supported on IBM Z (s390x) architectures. This section provide information about using this type of Elasticsearch cluster.

    * **[Referencing your own Elasticsearch](./documentation/Using-own-Elasticsearch.md)**

      Instead of using Elasticsearch as part of the stand-alone IBM Business Automation Workflow on containers deployment or the IBM Business Automation Workflow Server (or Workflow Runtime) in the IBM Cloud Pak for Business Automation deployment, you can decide to reference your own Elasticsearch. This section provide information about referencing this type of Elasticsearch cluster.

* **[Specifying Process Federation Server user authorizations on Kubernetes](./documentation/Authorizations.md)**

  When Process Federation Server is installed as part of the ICP4ACluster Custom Resource, the operator defines a default set of authorizations for Process Federation Server users, that you can customize depending on your needs. This section provides detailed information about that.

* **[Federating IBM Business Automation Workflow running on-premise](./documentation/Federating-on-premises-BAW.md)**

  You can configure IBM Process Federation Server containers to federate on-premise IBM Business Automation Workflow or IBM Business Process Manager systems. This section provides detailed information about that.

* **[Indexing case instances](./documentation/Indexing-Case-instances.md)**

  The case management tools provide support for indexing case instances in the Elasticsearch index. Full reindexing and live index updates are supported. This section provides detailed information about that.

* **[Migrating saved searches from Elasticsearch 6 to Elasticsearch 7](./documentation/Migrating-from-ES6-to-ES7.md)**

  In Process Federation Server V20.0.0.2, using an external Elasticsearch 6.x cluster is no longer supported. Instead, an external Elasticsearch 7.x (from 7.8.0) cluster is set up (see Elasticsearch product end of life dates at https://www.elastic.co/support/eol). This section provides detailed information about that.

* **[Maintaining, monitoring and troubleshooting IBM Process Federation Server in a container environment](./documentation/Maintaining-monitoring-and-troubleshooting.md)**

  You can maintain and monitor IBM Process Federation Server containers, and troubleshoot issues by following the procedures in this section.

  * **[Rebuilding federated systems indexes](./documentation/Rebuilding-indexes.md)**

    If you want to delete an Elasticsearch index containing data from a federated system, you can use the procedure documented in this section, that does not require you to restart any pod.
  
  * **[Monitoring IBM Process Federation Server](./documentation/Monitoring-PFS.md)**

    You can monitor a running server to gather information, detect issues, and perform actions without having to restart the server. This section provides detailed information about that.
  
  * **[Troubleshooting Process Federation Server in a container environment](./documentation/Troubleshooting-PFS.md)**

    If you face any issue with Process Federation Server, you can reach IBM support which will ask you to set up relevant logs and gather these logs and information about your configuration. This section provides detailed information about gathering logs and information for troubleshooting and support.


  
