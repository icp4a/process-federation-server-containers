# Administering and operating IBM Process Federation Server Containers

IBM® Process Federation Server is a component for IBM Business Automation Workflow (BAW) and IBM Business Process Manager (BPM) environments. This component creates a federated process environment that provides business users with a single point of access to their task list and launch list, regardless of the type of process that they are working on and the BAW and / or BPM back-end system on which the process artifacts are stored.

When IBM Business Automation Workflow traditional offering is deployed on-premises, IBM Process Federation Server is an optional component that is extensively documented in the [IBM Business Automation Workflow Knowledge Center](https://www.ibm.com/docs/en/baw/23.x?topic=server-traditional-installing-enabling-process-federation).

When IBM Business Automation Workflow containers are deployed on [Red Hat Openshift Container Platform](https://www.redhat.com/en/technologies/cloud-computing/openshift/container-platform) or other [CNCF Kubernetes platforms](https://www.cncf.io/projects/kubernetes/). Process Federation Server containers can also be deployed in order to provide a federated process environment that can be configured to federate on-premises Business Automation Workflow systems (V18.0.0.1 or later) with the Business Automation Workflow containers runtimes.
> Note: IBM Process Federation Server is an an optional component since 23.0.2. It was a by default deployed component prior 23.0.2.

For information about installing Process Federation Server on containers, see:
* for stand-alone IBM Business Automation Workflow on containers: [Installing Process Federation Server on containers](https://www.ibm.com/docs/en/baw/23.x?topic=containers-installing-process-federation-server);
* for IBM Business Automation Workflow deployment with IBM Cloud Pak for Business Automation: [Installing a CP4BA Process Federation Server deployment](https://www.ibm.com/docs/en/cloud-paks/cp-biz-automation/23.0.2?topic=deployments-installing-cp4ba-process-federation-server-production-deployment)

For both types of deployments, the Process Federation Server containers lifecycle is managed by the IBM Process Federation Server Operator that relies on a ProcessFederationServer Custom Resource for configuring the deployment. This Custom Resource is referenced as the _ProcessFederationServer Custom Resource_ in this Process Federation Server containers documentation.

The Process Federation Server Operator also has a dependency on a Custom Resource of the ICP4ACluster kind to retrieve shared configuration: This Custom Resource is referenced as the _ICP4ACluster Custom Resource_ in this Process Federation Server containers documentation.

> Note: Process Federation Server can be provisioned by defining `pfs_configuration` in ICP4ACluster Custom Resource prior 23.0.2. It can only be configured using ProcessFederationServer Custom Resource since 23.0.2.

## Documentation index

* **[Architecture of a Process Federation Server container environment](./documentation/Architecture.md)**

  The Process Federation Server Operator creates several kubernetes resources to deploy Process Federation Server. The topics in this section provide detailed information about those resources.
  
  * **[Process Federation Server statefulset](./documentation/PFS-Statefulset.md)**

    Process Federation Server is deployed as a [statefulset](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/) named `<cr-instance-name>-pfs`. This section provide detailed information about this statefulset, how to access its REST API and how to configure it.

  * **[Defining a federated data repository for Process Federation Server containers](./documentation/Defining-a-federated-data-repository.md)**

    Process Federation Server uses a [remote federated data respository](https://www.ibm.com/docs/en/baw/23.x?topic=service-declaring-federated-data-repository-in-serverxml), implemented with an Elasticsearch or Opensearch cluster, to store data. The topics in this section provide detailed information about the different types of Elasticsearch or Opensearch clusters that can be used by PFS containers to implement the federated data repository, depending on the type of IBM Business Automation Workflow containers installation:

    * **[Using an Elasticsearch cluster provided by IBM Cloud Pak foundational services](./documentation/Using-CPfs-Elasticsearch.md)**

      When deploying IBM Cloud Pak for Business Automation, an Elasticsearch cluster provided by IBM Cloud Pak foundational services is provisioned by default. This section provides information about using this type of Elasticsearch cluster as the Process Federation Server federated data repository.

    * **[Deploying an Opensearch cluster as part of the stand-alone IBM Business Automation Workflow on containers deployment](./documentation/Using-standalone-BAW-Opensearch.md)**

        When deploying stand-alone IBM® Business Automation Workflow on containers on AMD64 architectures, ProcessFederationServer Custom Resource is configured to create an Opensearch cluster as Federated Data Repository by PFS operator.

        

    * **[Referencing your own Elasticsearch or Opensearch cluster](./documentation/Using-own-Elasticsearch-or-Opensearch.md)**

      You can reference your own Elasticsearch or Opensearch cluster to use it as the remote federated data repository of your Process Federation Server deployment. This section provide information about referencing this type of clusters.

* **[Specifying Process Federation Server user authorizations on Kubernetes](./documentation/Authorizations.md)**

  When Process Federation Server is installed, the operator defines a default set of authorizations for Process Federation Server users, that you can customize depending on your needs. This section provides detailed information about that.

* **[Federating IBM Business Automation Workflow running on-premise](./documentation/Federating-on-premises-BAW.md)**

  You can configure IBM Process Federation Server containers to federate on-premise IBM Business Automation Workflow or IBM Business Process Manager systems. This section provides detailed information about that.

* **[Indexing case instances](./documentation/Indexing-Case-instances.md)**

  The case management tools provide support for indexing case instances in a federated data repository index. Full reindexing and live index updates are supported. This section provides detailed information about that.

* **[Migrating saved searches to a new federated data repository](./documentation/Migrating-Saved-Searches.md)**

  Process Federation Server Containers stores federated saved searches in an index of the federated data repository. To migrate from one  federated data repository to another, you need to migrate existing saved searches. This section provides detailed information about that.

* **[Maintaining, monitoring and troubleshooting IBM Process Federation Server in a container environment](./documentation/Maintaining-monitoring-and-troubleshooting.md)**

  You can maintain and monitor IBM Process Federation Server containers, and troubleshoot issues by following the procedures in this section.

  * **[Rebuilding federated systems indexes](./documentation/Rebuilding-indexes.md)**

    If you want to delete a federated data repository index containing data from a federated system, you can use the procedure documented in this section, that does not require you to restart any pod.
  
  * **[Monitoring IBM Process Federation Server](./documentation/Monitoring-PFS.md)**

    You can monitor a running server to gather information, detect issues, and perform actions without having to restart the server. This section provides detailed information about that.
  
  * **[Troubleshooting Process Federation Server in a container environment](./documentation/Troubleshooting-PFS.md)**

    If you face any issue with Process Federation Server, you can reach IBM support which will ask you to set up relevant logs and gather these logs and information about your configuration. This section provides detailed information about gathering logs and information for troubleshooting and support.


  
