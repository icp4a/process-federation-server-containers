# Specifying Process Federation Server user authorizations on Kubernetes

When Process Federation Server is installed as part of the ICP4ACluster Custom Resource, the operator defines a default set of authorizations for Process Federation Server users, that you can customize depending on your needs.

## Default authorizations for Process Federation Server users

The following attributes of the IBM® Cloud Pak for Business Automation Custom Resource define the administrator or the group of administrators to be used by the operator when setting the default authorizations:
* [pfs_configuration.admin_user_id](https://www.ibm.com/docs/en/cloud-paks/cp-biz-automation/22.0.1?topic=parameters-business-automation-workflow-runtime-workstream-services#ref_baw_params__pfserver): id(s) of the administrator user(s)
* [pfs_configuration.admin_group_id](https://www.ibm.com/docs/en/cloud-paks/cp-biz-automation/22.0.1?topic=parameters-business-automation-workflow-runtime-workstream-services#ref_baw_params__pfserver): id(s) of the administrator group(s)

These values are optional, consequently, by default there are no predefined administrator users or groups.
See [Mapping management roles for Liberty](https://www.ibm.com/docs/en/was-liberty/core?topic=liberty-mapping-management-roles)

The following table lists the default authorizations that are sets for Process Federation Server users when running in Kubernetes:

| Roles      | Users and groups |
| :---       | :---             |
| `administrator` | Admin users and/or the admin groups if they are defined in the Custom Resource |
| `restrictedBpmuser` | None |
| `bpmuser` | All authenticated users |
| `createSharedSavedSearch` | All authenticated users |
| `adminSavedSearch` | Admin users and/or the admin groups if they are defined in the Custom Resource |
| `adminSharedSavedSearch` | Admin users and/or the admin groups if they are defined in the Custom Resource |
| `doNotCreateSavedSearch` | None |

For more information about the different roles and the corresponding granted authorizations, see [Federated systems and authorization for saved searches](https://www.ibm.com/docs/en/baw/23.x?topic=portal-federated-systems-authorization-saved-searches).

## Defining your own set of authorizations

The syntax to reference a group or a user in authorizations differs depending on your deployment of Process Federation Server containers:

* For a deployment on OpenShift with Portal UI enabled, the syntax is `access-id="user:jwtrealm/USER_ID"` and `access-id="group:jwtrealm/GROUP_ID"`.
* For other deployments using UMS, the syntax is `access-id="user:o=defaultWIMFileBasedRealm/USER_ID"` and `access-id="group:o=defaultWIMFileBasedRealm/GROUP_ID"`.

Use the following procedure if you need to set up your own set of authorizations:

#### 1. Create a Kubernetes secret that contains a `.xml` file to be mounted as a configuration drop-in for the Process Federation Server server (for example, use the key _authorizations.xml_ in your secret):

  ```
kind: Secret
apiVersion: v1
metadata:
  name: <name-of-your-secret>
  namespace: <icp4ba-namespace>
type: Opaque
stringData:
  authorizations.xml: |-
    <?xml version="1.0" encoding="UTF-8"?>
    <server description="IBM Process Federation Server Config Dropin">
    <administrator-role>
        <user-access-id>[...]</user-access-id>
         [...]
   <group-access-id>[...]</group-access-id>
    </administrator-role>
    <authorization-roles id="com.ibm.bpm.federated.rest.authorization">
        <security-role name="restrictedBpmuser">
          [...]
        </security-role>
        <security-role name="bpmuser">
          [...]
        </security-role>
        <security-role name="createSharedSavedSearch">
          [...]
        </security-role>
        <security-role name="adminSharedSavedSearch">
          [...]
        </security-role>
        <security-role name="adminSavedSearch">
          [...]
        </security-role>
        <security-role name="doNotCreateSavedSearch">
          [...]
        </security-role>
    </authorization-roles>
  </server>
  ```
  For more information about the different roles and the corresponding granted authorizations, see [Federated systems and authorization for saved searches](https://www.ibm.com/docs/en/baw/23.x?topic=portal-federated-systems-authorization-saved-searches).

* To reference a user in the .xml file, and assign this user a role, use one of the following syntaxes:

  ```
  <user name="USER_NAME" access-id="user:jwtrealm/USER_ID" />
  ```

  ```
  <user name="USER_NAME" access-id="user:o=defaultWIMFileBasedRealm/USER_ID" />. 
  ```

  where `USER_NAME` and `USER_ID` are respectively the user's name and identifier.

* To reference a group, use one of the following syntaxes:

  ```
  <group name="GROUP_NAME" access-id="group:jwtrealm/GROUP_ID"/> 
  ```

  ```
  <group name="GROUP_NAME" access-id="group:o=defaultWIMFileBasedRealm/GROUP_ID"/>
  ```

  where `GROUP_NAME` and `GROUP_ID` are respectively the group's name and identifier.

* You can also use `<special-subject type="ALL_AUTHENTICATED_USERS"/>` entries to assign a role to all Process Federation Server users.

* The syntax for the administrator role slightly differs, and is documented in [Mapping management roles for Liberty](https://www.ibm.com/docs/en/was-liberty/base?topic=liberty-mapping-management-roles).

#### 2. Define the following values in the ICP4ACluster Custom Resource:

  ```
pfs_configuration:
    enable_default_security_roles: false
    config_dropins_overrides_secret: <name-of-your-secret>
  ```

  This step ensures that no default authorizations are defined for the Process Federation Server server by the ICP4ACluster Custom Resource, and that all the `.xml` files defined in the Kubernetes secret `<name-of-your-secret>`, including the one that defines the authorizations, will be mounted in the config dropins directory of the Process Federation Server server, and properly added to the Process Federation Server server configuration.
  
---

**Parent topic:** [Administering and operating IBM Process Federation Server](../README.md)

**Index:** [Documentation index](../README.md#documentation-index)
