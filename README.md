# Administering and operating IBM Process Federation Server Containers

[IBM Process Federation Server](https://www.ibm.com/docs/baw/25.0.0?topic=server-process-federation-components-overview) is a component for IBM Business Automation Workflow (BAW) environments. This component creates a federated process environment that provides business users with a single point of access to their list of tasks, process instances, case instances and launchable entities, regardless of the BAW  back-end system on which the artifacts are stored.

When IBM Business Automation Workflow traditional offering is deployed, IBM Process Federation Server is an optional component that is documented in the [IBM Business Automation Workflow Knowledge Center](https://www.ibm.com/docs/baw/25.0.0?topic=overview-process-federation-server).

When IBM Business Automation Workflow containers are deployed on [Red Hat Openshift Container Platform](https://www.redhat.com/en/technologies/cloud-computing/openshift/container-platform) or other [CNCF Kubernetes platforms](https://www.cncf.io/projects/kubernetes/). IBM Process Federation Server containers can also be deployed in order to provide a federated process environment that can be configured to federate on-premises IBM Business Automation Workflow systems (V18.0.0.1 or later) with the IBM Business Automation Workflow containers runtimes.
> Note: IBM Process Federation Server is an an optional component since 23.0.2. It was a by default deployed component prior 23.0.2.

For information about installing IBM Process Federation Server on containers, see:
* for stand-alone IBM Business Automation Workflow on containers: [Installing Process Federation Server for Business Automation Workflow on containers](https://www.ibm.com/docs/baw/25.0.0?topic=ic-installing-process-federation-server-business-automation-workflow-containers);
* for IBM Business Automation Workflow deployment with IBM Cloud Pak for Business Automation: [Installing a CP4BA IBM Process Federation Server deployment](https://www.ibm.com/docs/cloud-paks/cp-biz-automation/25.0.0?topic=deployments-installing-cp4ba-process-federation-server-production-deployment)

For both types of deployments, the IBM Process Federation Server containers lifecycle is managed by the IBM Process Federation Server operator that relies on a custom resource of the `ProcessFederationServer` kind for configuring the deployment or IBM Process Federation Server.

The IBM Process Federation Server operator also has a dependency on a custom resource of the `ICP4ACluster` kind to retrieve shared configuration.



## Documentation index

* **[Architecture of an IBM Process Federation Server container environment](./documentation/Architecture.md)**

  The IBM Process Federation Server operator creates several kubernetes resources to deploy IBM Process Federation Server. The topics in this section provide detailed information about those resources.
  
  * **[IBM Process Federation Server statefulset](./documentation/PFS-Statefulset.md)**

    IBM Process Federation Server is deployed as a [statefulset](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/) named `<cr-instance-name>-pfs`. This section provide detailed information about this statefulset, how to access its REST API and how to configure it.

  * **[Defining a Federated Data Repository for IBM Process Federation Server containers](./documentation/Defining-a-federated-data-repository.md)**

    IBM Process Federation Server uses a [remote Federated Data Repository](https://www.ibm.com/docs/baw/25.0.0?topic=server-process-federation-components-overview), implemented with an Elasticsearch or Opensearch cluster. The topics in this section provides detailed information about the different types of Elasticsearch or Opensearch clusters that can be used by IBM Process Federation Server containers to implement the Federated Data Repository:

    * **[Using an Opensearch cluster provided by IBM Cloud Pak foundational services](./documentation/Using-CPfs-Opensearch.md)**

      When deploying IBM Cloud Pak for Business Automation or stand-alone IBM Business Automation Workflow on containers, an Opensearch cluster provided by IBM Cloud Pak foundational services can provisioned. This section provides information about using this type of Opensearch cluster as the IBM Process Federation Server Federated Data Repository.

    * **[Referencing your own Elasticsearch or Opensearch cluster](./documentation/Using-own-Elasticsearch-or-Opensearch.md)**

      You can reference your own Elasticsearch or Opensearch cluster to use it as the remote Federated Data Repository of your IBM Process Federation Server deployment. This section provide information about referencing this type of clusters.

* **[Specifying IBM Process Federation Server user authorizations on Kubernetes](./documentation/Authorizations.md)**

  When IBM Process Federation Server is installed, the operator defines a default set of authorizations for IBM Process Federation Server users, that you can customize depending on your needs. This section provides detailed information about that.

* **[Federating a traditional IBM Business Automation Workflow](./documentation/Federating-traditional-BAW.md)**

  You can configure IBM Process Federation Server containers to federate traditional IBM Business Automation Workflow systems. This section provides detailed information about that.

* **[Indexing case instances](./documentation/Indexing-Case-instances.md)**

  The case management tools provide support for indexing case instances in a Federated Data Repository index. Full reindexing and live index updates are supported. This section provides detailed information about that.

* **[Migrating to a new Federated Data Repository](./documentation/Migrating.md)**

  When migrating to a new Federated Data Repository, you need to migrate existing saved searches and reusable queries. This section provides detailed information about that.

* **[Maintaining, monitoring and troubleshooting IBM Process Federation Server in a container environment](./documentation/Maintaining-monitoring-and-troubleshooting.md)**

  You can maintain and monitor IBM Process Federation Server containers, and troubleshoot issues by following the procedures in this section.

  * **[Rebuilding federated systems indexes](./documentation/Rebuilding-indexes.md)**

    If you want to delete a Federated Data Repository index containing data from a federated system, you can use the procedure documented in this section, that does not require you to restart any pod.
  
  * **[Monitoring IBM Process Federation Server](./documentation/Monitoring-PFS.md)**

    You can monitor a running server to gather information, detect issues, and perform actions without having to restart the server. This section provides detailed information about that.
  
  * **[Troubleshooting IBM Process Federation Server in a container environment](./documentation/Troubleshooting-PFS.md)**

    If you face any issue with IBM Process Federation Server, you can reach IBM support which will ask you to set up relevant logs and gather these logs and information about your configuration. This section provides detailed information about gathering logs and information for troubleshooting and support.


  
