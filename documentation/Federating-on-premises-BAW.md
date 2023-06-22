# Federating IBM Business Automation Workflow running on-premise

You can configure IBM® Process Federation Server containers to federate on-premise IBM Business Automation Workflow or IBM Business Process Manager systems.

The configuration of each on-premise system requires the following steps:
* [1- Preparing the on-premise Business Automation Workflow system for federation](#1--preparing-the-on-premise-business-automation-workflow-for-federation)
  * [1.1- Configuring Single Sign-On (SSO) with an on-premise system](#11--configuring-single-sign-on-sso-with-an-on-premise-system)
    * [1.1.1- Configuring SSO when federating from a standalone Business Automation Workflow deployment on containers](#111--configuring-sso-when-federating-from-a-standalone-business-automation-workflow-deployment-on-containers)
    * [1.1.2- Configuring SSO when federating from IBM Cloud Pak for Business Automation](#112--configuring-sso-when-federating-from-ibm-cloud-pak-for-business-automation)
  * [1.2- Enabling indexing on the on-premise Business Automation Workflow system](#12--enabling-indexing-on-the-on-premise-business-automation-workflow-system)
  * [1.3- Declaring Workplace as an allowed origin](#13--declaring-workplace-as-an-allowed-origin)
  * [1.4- Configuring the Content Security Policy on the on-premise system](#14--configuring-the-content-security-policy-on-the-on-premise-system)
* [2- Configuring Process Federation Server to federate the on-premise Business Automation Workflow system](#2--configuring-process-federation-server-to-federate-the-on-premise-business-automation-workflow-system)
  * [2.1- Planning your federation](#21--planning-your-federation)
  * [2.2- Creating a FederatedSystem custom resource to reference your on-premise system](#22--creating-a-federatedsystem-custom-resource-to-reference-your-on-premise-system)
    * [2.2.1- Basic configuration](#221--basic-configuration)
    * [2.2.2- Advanced configuration](#222--advanced-configuration)
    * [2.2.3- Mixing basic and advanced configurations](#223--mixing-basic-and-advanced-configurations)
* [3- Verifying that the on-premise Business Automation Workflow system is successfully federated](#3--verifying-that-the-on-premise-business-automation-workflow-system-is-successfully-federated)

> **Note:**
> If you upgraded from a version earlier than 23.0.1, and your topology was already federating an on-premise system before the upgrade, you must reconfigure the federation of the on-premise system by following the procedure provided in this document.

---

## 1- Preparing the on-premise Business Automation Workflow for federation

### 1.1- Configuring Single Sign-On (SSO) with an on-premise system

When a system is federated by Process Federation Server, Single Sign-On is required. When a Workplace user is successfully authenticated, different kinds of requests are issued to the on-premise federated system on the behalf of this Workplace user:
- PFS and Workplace issue queries that contain an `Authorization` HTTP header that holds a bearer token. 
- When working on a task or starting a new process instance on the on-premise federated system, the Client Side Human Service is rendered in an iframe. The requests from the iframe will not hold any `Authorization` HTTP header.

To validate these requests, the on-premise Business Automation Workflow system must be [configured as an OpenID Connect (OIDC) Relying Party](https://www.ibm.com/docs/en/was-nd/8.5.5?topic=users-configuring-openid-connect-relying-party).

The procedure is different depending on whether your Process Federation Server belongs to a [standalone Business Automation Workflow deployment on containers](#111--configuring-sso-when-federating-from-a-standalone-business-automation-workflow-deployment-on-containers) or an [IBM Cloud Pak® for Business Automation deployment](#112--configuring-sso-when-federating-from-ibm-cloud-pak-for-business-automation).

<br/>

#### 1.1.1- Configuring SSO when federating from a standalone Business Automation Workflow deployment on containers

If you are using a Process Federation Server that has been deployed as part of a standalone Business Automation Workflow deployment on containers, the OpenID Connect Provider that issues the token is the IBM User Management Service (UMS). The Business Automation Workflow system must be configured to use the same UMS as Process Federation Server for Single Sign-On (SSO), as documented in [Adding IBM Business Automation Workflow and IBM Federated Process Portal to use the User Management Service](https://www.ibm.com/docs/baw/23.x?topic=css-adding-business-automation-workflow-federated-process-portal-use-user-management-service).

> **Note:** Only Business Automation Workflow and Business Process Manager systems that provide UMS support can be federated by Process Federation Server running in Kubernetes as part of a standalone Business Automation Workflow deployment on containers.

<br/>

#### 1.1.2- Configuring SSO when federating from IBM Cloud Pak for Business Automation

If you are using a Process Federation Server that has been deployed as part of IBM Cloud Pak for Business Automation, the bearer tokens sent to the on-premise system in the `Authorization` HTTP header are issued by the Cloud Pak Platform UI (Zen) OpenID Connect Provider. Therefore, the incoming queries containing an Authorization HTTP header must be validated against the Cloud Pak Platform UI (Zen).

For the requests to Client Side Human Service originating from a Workplace iframe, user information is provided by a cookie issued by the Identity and Management (IM) service of the Cloud Pak and must therefore be validated against the Identity and Management (IM) service.

> **Note:** If the URL that is used in IBM Cloud Pak for Business Automation to access the on-premise system is pointing to a load balancer that distributes the load among the multiple nodes of your on-premise Business Automation Workflow cluster, **session affinity** must be guaranteed by the load balancer.

<br/>

##### 1.1.2.1- Retrieving the Zen URL

The location of this route (host and port) is needed in the remaining procedure. To retrieve the host and port, run the following command:
```
oc get route cpd
```

In the following example, the host for the `cpd` route is `cpd-myworkspace.apps.myocp.cp.mycompany.com` and the port is `443` (default https port).

<br/>

##### 1.1.2.2- Retrieving the Identity and Management service URL

The location of this route (host and port) is needed in the remaining procedure. To retrieve the host and port, run the following command:
```
oc get route cp-console
```

> **Note:** depending on your deployment the `cp-console` route can be either in the same namespace as your CP4BA deployment, or in the `ibm-common-services` namespace.

In the following example, the host for the `cp-console` route is `cp-console-myworkspace.apps.myocp.cp.mycompany.com` and the port is `443` (default https port).

<br/>
 
##### 1.1.2.3- Adding the signer certificates of the Zen and Identity and Management service URLs to the on-premise truststore

To add the signer certificate of the **cpd** route to the truststore of the on-premise Business Automation Workflow server:

1. Log in to the administrative console of the IBM WebSphere® Application server running on-premise Business Automation Workflow.
2. Expand `Security` and click `SSL certificate and key management`.
3. Under Configuration settings, click `Manage endpoint security configurations`.
4. Select the appropriate outbound configuration to get to the cell management scope.
5. Under `Related Items`, click `Key stores and certificates` and then click `CellDefaultTrustStore` key store.
6. Under `Additional Properties`, click `Signer certificates` and `Retrieve From Port`.
7. In the `Host` enter the host of the cpd route, in `Port` enter the port of the **cpd** route (443 by default), and in `Alias` enter a unique name.
8. Click `Retrieve Signer Information`.
9. Verify that the certificate information is for a certificate that you can trust.
10. Click `Apply` and `Save`.

Repeat the same procedure for the **cp-console** route.

<br/>

##### 1.1.2.4- Installing the OpenID Connect application on the on-premise system

The OpenId Connect application is shipped with your installation of on-premise Business Automation Workflow as an EAR file: _WebSphereOIDCRP.ear_. You must install this application by following step 1 of the procedure documented in [Configuring an OpenID Connect Relying Party](https://www.ibm.com/docs/en/was-nd/8.5.5?topic=users-configuring-openid-connect-relying-party).

<br/>

##### 1.1.2.5- Registering a new OIDC Client in Identity and Management service

To configure a Trust Association Interceptor (TAI), which acts as an OIDC Relying Party (RP) for Identity and Management (IM) service, you must first register in IM a client for your on-premise system. This registration can be performed automatically by [defining a CustomResourceDefinition (CRD) for OpenID Connect (OIDC) registration](https://www.ibm.com/docs/en/cpfs?topic=sign-automated-client-registration-method-3).

In the same namespace as the `cp-console` route, create the following Client custom resource: 
```
apiVersion: oidc.security.ibm.com/v1
kind: Client
metadata:
  name: onprem-client
  namespace: mynamespace
spec:
  oidcLibertyClient:
    redirect_uris:
      - >-
        https://myonprembaw.mycompany.com:9443/oidcclient/cp4ba_im_client
    post_logout_redirect_uris: []
    trusted_uri_prefixes: []
  secret: onprem-baw-admin-client-secret
```

Once the custom resource is created, inspect its yaml:
```
oc get client onprem-client -o=yaml
```

The `status` section should mention that the request was successful:
```
status:
  conditions:
  - lastTransitionTime: "2023-06-07T12:09:11Z"
    message: OIDC client registration successful
    reason: CreateClientSuccessful
    status: "True"
    type: Ready
```

You can now inspect the secret named `onprem-baw-admin-client-secret`, which contains the following 2 keys that are needed in the remaining procedure: `CLIENT_ID` and `CLIENT_SECRET`.

<br/>

##### 1.1.2.6- Configuring the TAI to authenticate incoming queries

You can now create the TAI with 2 providers:
- `provider_1` to authenticate against the Cloud Pak Platform UI (Zen) the requests from Workplace or PFS that come with an `Authorization` HTTP header that contains a 'Bearer' token
- `provider_2` to authenticate against the Identity and Management (IM) service, by following the OAuth flow, the requests from Workplace that do not come with an `Authorization` HTTP header that contains a 'Bearer' token

1. Log in to the administrative console of the WebSphere Application server running on-premise Business Automation Workflow. 
2. Expand `Security` and click `Global Security`.
3. In `User account repository`, make sure that your server is set up to use the same user repository as your deployment on Red Hat OpenShift, which is a prerequisite to enable federation of the system by the Process Federation Server that is deployed on Red Hat OpenShift.
4. In `Authentication`, click `Web and SIP security` and then click `Trust association`.
5. Under `Additional Properties`, click `Interceptors`.
6. Click `New` to add a `com.ibm.ws.security.oidc.client.RelyingParty` interceptor with the following custom properties in order to intercept all incoming queries coming from the Process Federation Server that is deployed on Red Hat OpenShift.

|Name|Value|Description|
|---|---|---|
|`provider_1.identifier`|`cp4ba_zen`| You can choose another identifier if desired.|
|`provider_1.filter`|`Authorization%=Bearer;X-PFS-ID~=^.*\|\|Authorization%=Bearer;Referer%=https://cpd-myworkspace.apps.myocp.cp.mycompany.com` | Filter only requests with an `Authorization` HTTP header that contains a 'Bearer' token and that have either an `X-PFS-ID` HTTP header (request from PFS) or a `Referer` HTTP header that is set on a URL from Workplace ([Zen URL](#1121--retrieving-the-zen-url)) |
|`provider_1.jwkEndpointUrl`|Location of the [cpd route](#1121--retrieving-the-zen-url), followed by `/auth/jwks` e.g.: `https://cpd-myworkspace.apps.myocp.cp.mycompany.com/auth/jwks`|Specifies the URL of the OIDC Provider's JSON Web Key (JWK) set document that contains the signing key required to validate the JSON Web Token (JWT) found in the `Authorization` HTTP header|
|`provider_1.useJwtFromRequest`|`required`| This provider requires a JWT for authentication. An OpenID Connect provider is not used.|
|`provider_1.issuerIdentifier`|`KNOXSSO`| Value expected for `iss` claim in the JWT issued by Zen|
|`provider_1.audiences`|`DSX`|Value expected for `aud` claim in the JWT issued by Zen|
|`provider_1.mapIdentityToRegistryUser`|`true`|The OpenID Connect RP maps the OpenID Connect authenticated user to the same user (by shortname) in the WebSphere Application Server user registry
|`provider_2.identifier`|`cp4ba_im_client` | You can choose another identifier if desired, but if doing so, you must adapt the `redirect_uris` property in the previous section|
|`provider_2.filter`|`Authorization!=Bearer;Referer%=https://cpd-myworkspace.apps.myocp.cp.mycompany.com\|\|Authorization!=Bearer;Referer%=https://myonprembaw.mycompany.com:9443` | Filter only requests without an `Authorization` HTTP header that contains a 'Bearer' token and with a `Referer` HTTP header which is either set on a URL from Workplace ([Zen URL](#1121--retrieving-the-zen-url)) or on a URL from the on-premise system |
|`provider_2.clientId`|Value of the `CLIENT_ID` key of the `onprem-baw-admin-client-secret` secret (see previous section)|ID that is used to identify the OpenID Connect RP instance to the OpenID connect Provider server|
|`provider_2.clientSecret`|Value of the `CLIENT_SECRET` key of the `onprem-baw-admin-client-secret` secret (see previous section)|Specifies the secret that is used by the OpenID Connect Provider to secure messages that are sent to this RP client in callback requests|
|`provider_2.discoveryEndpointUrl`|Location of the [cp-console route](#1122--retrieving-the-identity-and-management-service-url), followed by `/oidc/endpoint/OP/.well-known/openid-configuration` e.g.: `https://cp-console-myworkspace.apps.myocp.cp.mycompany.com/oidc/endpoint/OP/.well-known/openid-configuration`|Specifies the OpenID Connect Provider's discovery endpoint URL.|
|`provider_2.useRealm`|Set this property to the default WebSphere realm name, e.g.: `defaultWIMFileBasedRealm` | Specifies the realm name to be used for each request to this provider| 
|`provider_2.nonceEnabled`|`true`|When this property is set to true, a nonce parameter is sent to the OpenID Connect provider on the authentication request|  
|`provider_2.createSession`|`true`|Set this property to true if you want the runtime to create a new HTTP session for each client request. The property is required when you run in a cluster environment and you need the JSESSIONID cookie to maintain session affinity.|

<br/>

> **Note:**: The documentation about the OIDC RP custom properties is in [OpenID Connect Relying Party custom properties](https://www.ibm.com/docs/en/was-nd/8.5.5?topic=party-openid-connect-relying-custom-properties). The `provider_1.filter` and `provider_2.filter` properties can be fine-tuned if required. You may leverage:
> - the `Referer` HTTP Header that is present in all queries from Workplace or IM
> - the `X-PFS-ID` HTTP Header that is present in all queries issued by Process Federation Server and is set to `<namespace>-<ProcessFederationServer-custom-resource-name>`.
> The documentation about the `filter` property syntax is in [SAML web single sign-on (SSO) trust association interceptor (TAI) custom properties](https://www.ibm.com/docs/en/was/8.5.5?topic=swss-saml-web-single-sign-sso-trust-association-interceptor-tai-custom-properties)

<br/>

Once the interceptor configuration is applied and saved, restart the Business Automation Workflow server to activate the interceptor.

<br/>

> **Troubleshooting tip**: If the queries from Workplace or Process Federation Server return an HTTP 401 response, you can enable the following detailed traces in the WebSphere Application Server running on premise: `*=info:com.ibm.ws.security.oidc.*=all:com.ibm.ws.security.openidconnect.*=all:com.ibm.ws.security.openid20.*=all:com.ibm.ws.security.web.*=all`

<br/>
<br/>

### 1.2- Enabling indexing on the on-premise Business Automation Workflow system

The distributed Process Federation Server index enables process participants to see a consolidated list of BPD-related tasks from all Business Automation Workflow systems in the federated environment. For data from a federated Business Automation Workflow system to be included in the index, you must enable indexing on the Business Automation Workflow system, as described in [Enabling indexing on a federated system](https://www.ibm.com/docs/baw/23.x?topic=systems-enabling-indexing-federated-system).

As part of this configuration:
- you must create the change log tables in the Process Server database of the Business Automation Workflow system. In the pod that runs Process Federation Server, the database scripts are located in `/opt/ibm/wlp/ibmProcessFederationServer/wlp-ext/dbscripts/`. To retrieve these scripts, execute the following command:
```
oc cp <PFS_pod_name>:/opt/ibm/wlp/ibmProcessFederationServer/wlp-ext/dbscripts/ .
```
- you must update the `<BAW_root>/profiles/<Dmgr_Profile>/config/cells/<Cell_name>/nodes/<Node_name>/servers/<server_name>/process-server/config/100Custom.xml` file of each node of your cluster so that it contains the following configuration:
```
</properties> 
    <server merge="mergeChildren">
        <search-index merge="mergeChildren">
            <federated-index-enabled merge="replace">true</federated-index-enabled>
        </search-index>
    </server>
</properties> 
```

<br/>

### 1.3- Declaring Workplace as an allowed origin

<br/>

To allow Workplace to send queries to the on-premise system REST API, Workplace URL must be declared as an allowed origin on the on-premise system configuration. To achieve this, merge the following configuration in the same `100Custom.xml` file(s) as in the previous section:
```
</properties> 
    <server merge="mergeChildren">
        <rest merge="mergeChildren">
            <allowed-origins>https://cpd-myworkspace.apps.myocp.cp.mycompany.com</allowed-origins>
        </rest>
    </server>
</properties> 
```

> Note: After manually editing the `100Custom.xml` files, you must trigger a full resynchronization of the nodes: 
> 1. In the on-premise WebSphere Application Server administrative console, click **System administration > Nodes**.
> 2. Select all the nodes and click **Full Resynchronize**.
> 3. Go to **Servers > Server Types > WebSphere application servers**.
> 4. Select the BAW server and click **Restart**.

<br/>

### 1.4- Configuring the Content Security Policy on the on-premise system

The Content Security Policy must be properly configured to allow Workplace to work on tasks and process instances from the on-premise system. 
The procedure is different depending on whether your Process Federation Server belongs to an [IBM Cloud Pak for Business Automation deployment](#141--configuring-the-content-security-policy-when-federating-from-ibm-cloud-pak-for-business-automation) or a [standalone Business Automation Workflow deployment on containers](#142--configuring-the-content-security-policy-when-federating-from-a-standalone-business-automation-workflow-deployment-on-containers).

<br/>

#### 1.4.1- Configuring the Content Security Policy when federating from IBM Cloud Pak for Business Automation

When federating an on-premise system from IBM Cloud Pak for Business Automation, for Workplace to render the Client Side Human Services without hitting violations of the on-premise Content Security Policy, the on-premise system URL must be declared as a valid source for the following directives:
 - `default-src`
 - `script-src`
 - `frame-src`
 - `object-src`
 - `connect-src`
 - `img-src`
 - `style-src`
 - `font-src`

The Workplace URL must also be declared in the `frame-ancestors` directive.

The following example shows how to set the on-premise Content Security Policy as appropriate:
```
# ./wsadmin.sh -conntype SOAP -port 8879 -host localhost -user bpmadmin -password Passw0rd -lang jython
WASX7209I: Connected to process "dmgr" on node Dmgr01 using SOAP connector;  The type of process is: DeploymentManager
WASX7031I: For help, enter: "print Help.help()"

wsadmin>print AdminTask.setBPMProperty(['-de', 'DE1', '-name', 'Security.ContentSecurityPolicyHeaderValue', '-value', "default-src data: blob: filesystem: about: ws: wss: 'unsafe-inline' 'unsafe-eval' https://myonprembaw.mycompany.com:9443 ; script-src 'unsafe-inline' 'unsafe-eval' https://myonprembaw.mycompany.com:9443 ; frame-src https://myonprembaw.mycompany.com:9443 ; object-src  https://myonprembaw.mycompany.com:9443 ; connect-src 'unsafe-inline' https://myonprembaw.mycompany.com:9443 ; img-src data: blob: 'unsafe-inline' https://myonprembaw.mycompany.com:9443 ; style-src data: blob: 'unsafe-inline' https://myonprembaw.mycompany.com:9443 ; font-src data: blob: 'unsafe-inline' https://myonprembaw.mycompany.com:9443 ; frame-ancestors https://cpd-myworkspace.apps.myocp.cp.mycompany.com"])
true
wsadmin>AdminConfig.save()
''
```


> **Note:** The documentation about the Security-hardening properties is [here](https://www.ibm.com/docs/en/baw/23.x?topic=environment-security-hardening-properties).

<br/>

#### 1.4.2- Configuring the Content Security Policy when federating from a standalone Business Automation Workflow deployment on containers

When federating an on-premise system from a standalone IBM® Business Automation Workflow deployment on containers, for Workplace to render the Client Side Human Services without hitting violations of the Content Security Policies, you must edit the **ICP4ACluster** custom resource as follows:
1. In the **ICP4ACluster** custom resource, locate the item in `spec.baw_configuration` where you have `host_federated_portal: true`.
2. In this item, merge the following properties:
```
    host_federated_portal: true
    federated_portal:
      content_security_policy_additional_origins:
        - 'https://myonprembaw.mycompany.com:9443'
```

You must also configure the on-premise Content Security Policy to reference the URLS to PFS, Workplace, UMS and Application Engine in the `connect-src`, `frame-src`and `script-sec-elem` directives.

To retrieve these URLs, when running on Red Hat OpenShift, you can issue these commands, where `<cr-name>` is the name of the **ICP4Acluster** CR and `<baw-instance-name>` is the value of the `name` property of the item in `spec.baw_configuration` where you have `host_federated_portal: true`:
```
oc get route <cr-name>-pfs-route
oc get route <cr-name>-<baw-instance-name>-baw-server
oc get route <cr-name>-ums-sso-route
oc get route <cr-name>-workspace-aae-ae-service
```

Here is an example of how to set the on-premise Content Security Policy as appropriate:
```
# ./wsadmin.sh -conntype SOAP -port 8879 -host localhost -user bpmadmin -password Passw0rd -lang jython
WASX7209I: Connected to process "dmgr" on node Dmgr01 using SOAP connector;  The type of process is: DeploymentManager
WASX7031I: For help, enter: "print Help.help()"

wsadmin>print AdminTask.setBPMProperty(['-de', 'DE1', '-name', 'Security.ContentSecurityPolicyHeaderValue', '-value', "default-src 'self' 'unsafe-inline' 'unsafe-eval'; connect-src 'self' ums-sso-mynamespace.apps.myocp.mycompany.com pfs-mynamespace.apps.myocp.mycompany.com ae-workspace-mynamespace.apps.myocp.mycompany.com baw-bawins1-mynamespace.apps.myocp.mycompany.com ; frame-src 'self' ums-sso-mynamespace.apps.myocp.mycompany.com pfs-mynamespace.apps.myocp.mycompany.com ae-workspace-mynamespace.apps.myocp.mycompany.com baw-bawins1-mynamespace.apps.myocp.mycompany.com ; script-src-elem 'self' 'unsafe-inline' 'unsafe-eval' ums-sso-mynamespace.apps.myocp.mycompany.com pfs-mynamespace.apps.myocp.mycompany.com ae-workspace-mynamespace.apps.myocp.mycompany.com baw-bawins1-mynamespace.apps.myocp.mycompany.com ; font-src 'self' https://fonts.gstatic.com ; img-src 'self' data: blob:"])
true
wsadmin>AdminConfig.save()
''
```

> **Note:** The documentation about the Security-hardening properties is [here](https://www.ibm.com/docs/en/baw/23.x?topic=environment-security-hardening-properties).

<br/>

---

## 2- Configuring Process Federation Server to federate the on-premise Business Automation Workflow system

### 2.1- Planning your federation

Now that the Business Automation Workflow system is properly configured to be federated, you have two options regarding how data will be indexed from the on-premise relational database to the Federated Data Repository (Elasticsearch/Opensearch):
- Configure the Process Federation Server running on Openshift/Kubernetes to declare the federated system, a retriever, and also an indexer to index data from the on-premise database to the federated system into the Federated Data Repository already used by Process Federation Server running on Openshift/Kubernetes.
- Configure a regular on-premise Process Federation Server deployment that will perform indexing from the on-premise database to the Federated Data Repository already used by Process Federation Server running on Openshift/Kubernetes. With this approach, the Process Federation Server running on Openshift/Kubernetes is not responsible for indexing data from the on-premise federated system and will only declare the federated system and a retriever to the on-premise federated system. This approach provides better performance if there is a network latency larger than 10ms between the on-premises site and Process Federation Server running on Openshift/Kubernetes.

![Indexing options diagram](/documentation/images/indexing-options.png)

<br/>

### 2.2- Creating a FederatedSystem custom resource to reference your on-premise system

From version 23.0.1, the Process Federation Server operator uses a new Custom Resource Definition named "**FederatedSystem**" to seamlessly federate Workflow and Business Process Management systems. For each system to federate, an instance of **FederatedSystem** custom resource must be instantiated. This instantiation of **FederatedSystem** custom resource is performed internally by the operators when federating systems that are deployed on your Openhift/Kubernetes cluster. But to federate an on-premise Business Automation Workflow system, you must manually create a **FederatedSystem** custom resource.

You can choose between the following options when configuring the **FederatedSystem** custom resource for your on-premise federated system:
- [Basic configuration](#221--basic-configuration): provide a minimum set of settings, and let the Process Federation Server operator generate the [Liberty configuration dropin](https://www.ibm.com/docs/en/was-liberty/base?topic=files-using-configuration-dropins-folder-specify-server-configuration) to federate the on-premise system.
- [Advanced configuration](#222--advanced-configuration): provide your own [Liberty configuration dropin](https://www.ibm.com/docs/en/was-liberty/base?topic=files-using-configuration-dropins-folder-specify-server-configuration) that configures Process Federation Server to federate the on-premise system.
- [Mixing basic and advanced configurations](#223--mixing-basic-and-advanced-configurations): use [basic federation](#221--basic-configuration) to provide a minimum set of settings and let the Process Federation Server operator generate the [Liberty configuration dropin](https://www.ibm.com/docs/en/was-liberty/base?topic=files-using-configuration-dropins-folder-specify-server-configuration), and use [advanced configuration](#222--advanced-configuration) to provide your own [Liberty configuration dropin](https://www.ibm.com/docs/en/was-liberty/base?topic=files-using-configuration-dropins-folder-specify-server-configuration) that overrides or augments the generated configuration.

<br/>

#### 2.2.1- Basic configuration

The basic configuration is the most straightforward approach: you provide the base URL to the on-premise system, the corresponding credentials and TLS certificate, and the operator takes care of retrieving all required data from the system to configure Process Federation Server to federate the on-premise federated system. 

If you also want Process Federation Server to index data from the on-premise database to the federated data repository, some additional properties about the on-premise database are also required to [enable indexing](#2214--enabling-indexing).

##### 2.2.1.1- Creating a secret with credentials to the federated system

When using the basic configuration, the Process Federation Server operator needs to perform a query against the `/rest/bpm/wle/v1/systems` REST API exposed by the on-premise Business Automation Workflow system. The credentials used to authenticate this request are sensitive information and must be packaged in a secret. This secret must contain the following keys: `serverUser` and `serverPassword`.

For example:
```
kubectl create secret generic onprem-credential --from-literal=serverUser=user --from-literal=serverPassword=password
```

<br/>

##### 2.2.1.2- Creating a secret with the TLS certificate

You must also create a Kubernetes secret that contains the on-premise Business Automation Workflow system root certificate that will later be added to the Process Federation Server truststore.

To get the on-premise system root certificate, follow these steps:

1. In the on-premise WebSphere Application Server administrative console, click **Security > SSL certificate and key management > Key stores and certificates > CellDefaultTrustStore > Signer certificates**.
2. Select the root certificate and click **Extract**.
3. Name the file. For example, name the file `on-prem-cert.crt`.
4. For the data type, select Base64-encoded ASCII data.
5. Click **Apply**. From the message, note where the certificate is stored on the file system. Then transfer it to the cloud platform where Process Federation Server is deployed.
6. You can create the Kubernetes secret as shown in the following example:
```
kubectl create secret generic onprem-cert-secret --from-file=tls.crt=/path/to/on-prem-cert.crt
```

<br/>

##### 2.2.1.3- Creating the FederatedSystem custom resource for basic configuration
You can now create the **FederatedSystem** custom resource.

Here is an example of a **FederatedSystem** custom resource that does not declare an indexer, assuming that indexing is performed by a Process Federation Server running on premisde that indexes in the same Elasticsearch cluster as the one used by the Process Federation Server running on Openshift/Kubernetes:

```
apiVersion: icp4a.ibm.com/v1
kind: FederatedSystem
metadata:
  name: onprem-baw-federatedsystem
  namespace: demo-project
spec:
  url: https://onprem-baw:9443/
  credentialSecret: onprem-credential
  certificates:
    secrets:
      - onprem-cert-secret
    
```

The complete documentation of the **FederatedSystem** custom resource properties can be found in [Federated system parameters](https://www.ibm.com/docs/SSYHZ8_23.0.1/com.ibm.dba.install/op_topics/ref_fs_params.html).

<br/>

##### 2.2.1.4- Enabling indexing

If you plan to also have Process Federation Server running on Openshift/Kubernetes to perform the indexing of the on-premise Business Automation Workflow tasks and process instances into the Federated Data repository, you must add a section in the **FederatedSystem** custom resource specification to provide details related to the database used by the on-premise system.

The supported indexer datasources are:
- DB2
- PostgreSQL
- Oracle
- Microsoft SQL Server

Depending on the database used, you must provide the corresponding section. For example, for a DB2 database, you must add a section `spec.db2Indexer` as documented in [Federated system parameters](https://www.ibm.com/docs/SSYHZ8_23.0.1/com.ibm.dba.install/op_topics/ref_fs_params.html).

When the database is accessed with an SSL connection, you must also package in a secret the TLS public key certificate to add to the Process Federation Server truststore for the SSL connection to the database to be authorized, similarly to what is documented [here](#2212--create-a-secret-with-the-tls-certifcate-to-trust). Then, reference this secret in the **FederatedSystem** custom resource specification. For example, for a DB2 indexer, the certificate would have to be referenced under the key:  `spec.db2Indexer.sslCertSecret`.


<br/>
<br/>

#### 2.2.2- Advanced configuration

##### 2.2.2.1- Creating a secret with the Liberty configuration dropin
Instead of using the basic federation, you can use the advanced federation where you will have the responsibilty for creating a secret that holds keys to be mounted in the `/configDropins/overrides` folder of the pod running Process Federation Server.

You must create an entry in this secret that contains a key that ends with ".xml". This entry contains the following configuration elements:

* the declaration of the `<ibmPfs_federatedSystem>` element
* the datasource of the on-premise Business Automation Workflow system (`<datasource>` element)
* the `<ibmPfs_bpdRetriever>` element
* the `<ibmPfs_bpdIndexer>` element if indexing is to be performed from Process Federation Server running on Openshift/Kubernetes

You can refer to [Configuration properties for federated systems](https://www.ibm.com/docs/baw/23.x?topic=systems-configuration-properties-federated) for detailed information about the above configuration elements.

When configuring the `<ibmPfs_federatedSystem>`, you must take care that the `restUrlPrefix` and `taskCompletionUrlPrefix` properties are referencing the Process Federation Server proxy API URLs to reach the on-premise system. Only the `internalRestUrlPrefix` property of the `<ibmPfs_bpdRetriever>` must reference the direct URL to the federated system.

For example:
```
    <ibmPfs_federatedSystem id="on-prem-system-1"
        restUrlPrefix="https://cpd-myworkspace.apps.myocp.cp.mycompany.com/pfs/rest/bpm/federated/v1/proxy/on-prem-system-1/rest/bpm/wle" 
        taskCompletionUrlPrefix="https://cpd-myworkspace.apps.myocp.cp.mycompany.com/pfs/rest/bpm/federated/v1/proxy/on-prem-system-1/teamworks"
        [...]
    />
    <ibmPfs_bpdRetriever id="on-prem-system-1-retriever"
        federatedSystemRef="on-prem-system-1"
        internalRestUrlPrefix="https://onprem-host:9443/rest/bpm/wle"
        [...]
    />
```
<br/>

> **Note:**
> Process Federation Server containers has a set of predefined libraries that you can reference when you declare the jdbcDriver used by your datasource:
> * For a Db2 datasource, use `<jdbcDriver libraryRef="DB2JCC4Lib"/>`
> * For a PostgreSQL datasource, use `<jdbcDriver libraryRef="postgresqlJDBCLib"/>`
> * For an Oracle datasource, use `<jdbcDriver libraryRef="OracleLib"/>`
> * For a Microsoft SQL Server datasource, use `<jdbcDriver libraryRef="MSJDBCLib"/>`

<br/>

##### 2.2.2.2- Creating the secrets to set up the truststore

To allow SSL connection to the on-premise federated system, and to the on-premise database, you must ensure that TLS certificates are added to the IBM® Process Federation Server truststore by packaging them into secrets similarly to what is documented [here](#2212--create-a-secret-with-the-tls-certifcate-to-trust), and then reference this secret in `spec.certificates.secrets`.

<br/>

##### 2.2.2.3- Creating the FederatedSystem custom resource for advanced federation

Once this secret is created, you can create the **FederatedSystem** custom resource.

The following example shows a FederatedSystem custom resource that uses the advanced federation:

```
apiVersion: icp4a.ibm.com/v1
kind: FederatedSystem
metadata:
  name: onprem-baw-federatedsystem
  namespace: demo-project
spec:
  advancedConfig: onprem-baw-config-secret
  certificates:
    secrets:
      - onprem-cert-secret
      - database-cert-secret
    
```

<br/>
<br/>

#### 2.2.3- Mixing basic and advanced configurations

It is possible to set up a **FederatedSystem** custom resource by following the [basic configuration](#221--basic-configuration) procedure - so that the Process Federation Server operator generates the [Liberty configuration dropin](https://www.ibm.com/docs/en/was-liberty/base?topic=files-using-configuration-dropins-folder-specify-server-configuration) that will declare the federated system - and also provide a secret referenced as `spec.advancedConfig` as described in [advanced configuration](#222--advanced-configuration). 

This secret would contain [Liberty configuration dropins](https://www.ibm.com/docs/en/was-liberty/base?topic=files-using-configuration-dropins-folder-specify-server-configuration):
- to declare new configuration tags like `<ibmPfs_launchableEntity>` as documented [here](https://www.ibm.com/docs/baw/23.x?topic=portal-dashboards-processes-services)
- to override the configuration tags which are generated using the [Basic federation](#221--basic-federation) procedure. To achieve this, you must first copy-paste the content of the key from the secret with the same name as the **FederatedSystem** custom resource into a new key in the secret referenced as `spec.advancedConfig`, and then adapt it as needed to add or change the default configuration properties. For the generated configuration to be overridden by your custom configuration, you must take care that the key in the secret referenced as `spec.advancedConfig` is alphabetically greater than the key in the secret with the same name as the **FederatedSystem** custom resource. The `id` property of the generated configuration tags must remain unchanged.


---

## 3- Verifying that the on-premise Business Automation Workflow system is successfully federated

Once the **FederatedSystem** custom resource is created, you can verify that the system is successfully federated by calling the following Process Federation Server API:
`/rest/bpm/federated/v1/system`.

In the response, verify that in the `federationResult` array, there is an item for your on-premise federated system and that the `statusCode` value of this item is `"200"`.

For more details about how to access the Process Federation Server REST API, see [this documentation](/documentation/PFS-Statefulset.md#accessing-the-rest-api).

--- 

**Parent topic:** [Administering and operating IBM® Process Federation Server](../README.md)

**Index:** [Documentation index](../README.md#documentation-index)
