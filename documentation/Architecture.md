# Architecture of a Process Federation Server container environment

When Process Federation Server is deployed using a ProcessFederationServer Custom Resource, several kubernetes resources are created. The topics in this section provide detailed information about those resources.

* **[Process Federation Server statefulset](./PFS-Statefulset.md)**

    Process Federation Server is deployed as a [statefulset](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/) named `<cr-instance-name>-pfs`.

* **[Defining a federated data repository for Process Federation Server containers](./Defining-a-federated-data-repository.md)**

    Process Federation Server uses a [remote federated data respository](https://www.ibm.com/docs/en/baw/23.x?topic=service-declaring-federated-data-repository-in-serverxml), implemented with an Elasticsearch or Opensearch cluster, to store data. 


---
**Parent topic:** [Administering and operating IBM Process Federation Server Containers](../README.md)

**Index:** [Documentation index](../README.md#documentation-index)
