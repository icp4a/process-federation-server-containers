# Federating a traditional IBM Business Automation Workflow system from IBM Process Federation Server running on containers

You can configure IBM Process Federation Server containers to federate a traditional IBM Business Automation Workflow.

The configuration requires the following steps:
- [1- Preparing the traditional IBM Business Automation Workflow system for federation](#1--preparing-the-traditional-ibm-business-automation-workflow-system-for-federation)
  - [1.1- Configuring Single Sign-On (SSO) with a traditional IBM Business Automation Workflow system](#11--configuring-single-sign-on-sso-with-a-traditional-ibm-business-automation-workflow-system)
    - [1.1.1- Retrieving the Zen URL](#111--retrieving-the-zen-url)
    - [1.1.2- Retrieving the Identity and Management service URL](#112--retrieving-the-identity-and-management-service-url)
    - [1.1.3- Adding the signer certificates of the Zen and Identity and Management service URLs to the traditional IBM Business Automation Workflow truststore](#113--adding-the-signer-certificates-of-the-zen-and-identity-and-management-service-urls-to-the-traditional-ibm-business-automation-workflow-truststore)
    - [1.1.4- Installing the OpenID Connect application on the traditional IBM Business Automation Workflow system](#114--installing-the-openid-connect-application-on-the-traditional-ibm-business-automation-workflow-system)
    - [1.1.5- Registering a new OIDC Client in Identity and Management service](#115--registering-a-new-oidc-client-in-identity-and-management-service)
    - [1.1.6- Configuring the TAI to authenticate incoming queries](#116--configuring-the-tai-to-authenticate-incoming-queries)
    - [1.1.7- Give the OIDC TAI a higher priority than the PreemptiveBasicAuthTAI](#117--give-the-oidc-tai-a-higher-priority-than-the-preemptivebasicauthtai)
    - [1.1.8- Expected authentication flows](#118--expected-authentication-flows)
      - [1.1.8.1- For requests with an Authorization Bearer token](#1181--for-requests-with-an-authorization-bearer-token)
      - [1.1.8.2- For requests without Authorization Bearer token](#1182--for-requests-without-authorization-bearer-token)
  - [1.2- Enabling indexing on the traditional IBM Business Automation Workflow system](#12--enabling-indexing-on-the-traditional-ibm-business-automation-workflow-system)
    - [1.2.1- Enabling the process indexing](#121--enabling-the-process-indexing)
      - [1.2.1.1- IBM Business Automation Workflow 24.0.0.0 and newer](#1211--ibm-business-automation-workflow-24000-and-newer)
      - [1.2.1.2- IBM Business Automation Workflow 23.0.2 and older](#1212--ibm-business-automation-workflow-2302-and-older)
    - [1.2.2- Indexing case instances](#122--indexing-case-instances)
  - [1.3- Declaring Workplace as an allowed origin](#13--declaring-workplace-as-an-allowed-origin)
    - [1.3.1- Configure BAW for allowed origins](#131--configure-baw-for-allowed-origins)
    - [1.3.2- Configure any frontend IHS loadbalancer for allowed origins](#132--configure-any-frontend-ihs-loadbalancer-for-allowed-origins)
  - [1.4- Configuring the Content Security Policy on the traditional IBM Business Automation Workflow system](#14--configuring-the-content-security-policy-on-the-traditional-ibm-business-automation-workflow-system)
- [2- Configuring IBM Process Federation Server to federate the traditional IBM Business Automation Workflow system](#2--configuring-ibm-process-federation-server-to-federate-the-traditional-ibm-business-automation-workflow-system)
  - [2.1- Planning your federation](#21--planning-your-federation)
    - [2.1.1- IBM Business Automation Workflow 24.0.0.0 and newer](#211--ibm-business-automation-workflow-24000-and-newer)
    - [2.1.2- IBM Business Automation Workflow 23.0.2 and older](#212--ibm-business-automation-workflow-2302-and-older)
  - [2.2- Creating a FederatedSystem custom resource to reference your traditional IBM Business Automation Workflow system](#22--creating-a-federatedsystem-custom-resource-to-reference-your-traditional-ibm-business-automation-workflow-system)
    - [2.2.1- Basic configuration](#221--basic-configuration)
      - [2.2.1.1- Creating a secret with credentials to the federated system](#2211--creating-a-secret-with-credentials-to-the-federated-system)
      - [2.2.1.2- Creating a secret with the TLS certificate](#2212--creating-a-secret-with-the-tls-certificate)
      - [2.2.1.3- Creating the FederatedSystem custom resource for basic configuration](#2213--creating-the-federatedsystem-custom-resource-for-basic-configuration)
      - [2.2.1.4- Enabling indexing](#2214--enabling-indexing)
    - [2.2.2- Advanced configuration](#222--advanced-configuration)
      - [2.2.2.1- Creating a secret with the Liberty configuration dropin](#2221--creating-a-secret-with-the-liberty-configuration-dropin)
      - [2.2.2.2- Creating the secrets to set up the truststore](#2222--creating-the-secrets-to-set-up-the-truststore)
      - [2.2.2.3- Creating the FederatedSystem custom resource for advanced federation](#2223--creating-the-federatedsystem-custom-resource-for-advanced-federation)
    - [2.2.3- Mixing basic and advanced configurations](#223--mixing-basic-and-advanced-configurations)
- [3- Verifying that the traditional IBM Business Automation Workflow system is successfully federated](#3--verifying-that-the-traditional-ibm-business-automation-workflow-system-is-successfully-federated)


> **Note:**
> If you upgraded from a version earlier than 23.0.1, and your topology was already federating a traditional IBM Business Automation Workflow system before the upgrade, you must reconfigure the federation of the traditional IBM Business Automation Workflow system by following the procedure provided in this document.

<br/>

---

## 1- Preparing the traditional IBM Business Automation Workflow system for federation

### 1.1- Configuring Single Sign-On (SSO) with a traditional IBM Business Automation Workflow system

When a system is federated by IBM Process Federation Server, Single Sign-On is required. When a Workplace user is successfully authenticated, different kinds of requests are issued to the traditional IBM Business Automation Workflow federated system on the behalf of this Workplace user:
- IBM Process Federation Server and Workplace issue queries that contain an `Authorization` HTTP header that holds a bearer token. 
- When working on a task or starting a new process instance on the traditional IBM Business Automation Workflow federated system, the Client Side Human Service is rendered in an iframe. The requests from the iframe will not hold any `Authorization` HTTP header.

To validate these requests, the traditional IBM Business Automation Workflow system must be [configured as an OpenID Connect (OIDC) Relying Party](https://www.ibm.com/docs/en/was-nd/8.5.5?topic=users-configuring-openid-connect-relying-party).
<br/>

The bearer tokens sent to the traditional IBM Business Automation Workflow system in the `Authorization` HTTP header are issued by the Cloud Pak Platform UI (Zen) OpenID Connect Provider. Therefore, the incoming queries containing an Authorization HTTP header must be validated against the Cloud Pak Platform UI (Zen).

For the requests to Client Side Human Service originating from a Workplace iframe, user information is provided by a cookie issued by the Identity and Management (IM) service of the Cloud Pak and must therefore be validated against the Identity and Management (IM) service.

> **Note:** If the URL that is used to access the traditional IBM Business Automation Workflow system is pointing to a load balancer that distributes the load among the multiple nodes of your traditional IBM Business Automation Workflow cluster, **session affinity** must be guaranteed by the load balancer.

<br/>

#### 1.1.1- Retrieving the Zen URL

The location of the host and port to Zen is needed in the remaining procedure.

If you are using an IBM Process Federation Server instance deployed on Openshift, run the following command:
```
oc get route cpd
```

If you are using an IBM Process Federation Server instance deployed on a Kubernetes CNCF platform which is not Openshift, run the following command:
```
kubectl get ingress zen-ingress
```

In the following example, the host for the `cpd` route is `cpd-myworkspace.apps.myocp.cp.mycompany.com` and the port is `443` (default https port).

<br/>

#### 1.1.2- Retrieving the Identity and Management service URL

The location of the host and port to the Identity and Management (IM) service is needed in the remaining procedure.

If you are using an IBM Process Federation Server instance deployed on Openshift, run the following command:
```
oc get route cp-console
```

If you are using an IBM Process Federation Server instance deployed on a Kubernetes CNCF platform which is not Openshift, run the following command:
```
kubectl get ingress cncf-platform-oidc
```

> **Note:** depending on your deployment, the `cp-console` route can be either in the same namespace as your CP4BA deployment, or in the `ibm-common-services` namespace.

In the following example, the host for the `cp-console` route is `cp-console-myworkspace.apps.myocp.cp.mycompany.com` and the port is `443` (default https port).

<br/>
 
#### 1.1.3- Adding the signer certificates of the Zen and Identity and Management service URLs to the traditional IBM Business Automation Workflow truststore

To add the signer certificate of the **cpd** route to the truststore of the traditional IBM Business Automation Workflow server:

1. Log in to the administrative console of the IBM WebSphere® Application server running traditional IBM Business Automation Workflow.
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

#### 1.1.4- Installing the OpenID Connect application on the traditional IBM Business Automation Workflow system

The OpenId Connect application is shipped with your installation of the traditional IBM Business Automation Workflow system as an EAR file: _WebSphereOIDCRP.ear_. You must install this application by following **steps 1 and 2** of the procedure documented in [Configuring an OpenID Connect Relying Party](https://www.ibm.com/docs/en/was-nd/8.5.5?topic=users-configuring-openid-connect-relying-party).

<br/>

#### 1.1.5- Registering a new OIDC Client in Identity and Management service

> **Note** : This step is only required if SSO is not already configured in some way (SpNego, SAML)

To configure a Trust Association Interceptor (TAI), which acts as an OIDC Relying Party (RP) for Identity and Management (IM) service, you must first register in IM a client for your traditional IBM Business Automation Workflow system. This registration can be performed automatically by [defining a CustomResourceDefinition (CRD) for OpenID Connect (OIDC) registration](https://www.ibm.com/docs/en/cpfs?topic=sign-automated-client-registration-method-3).

In the same namespace as the `cp-console` route, create the following Client custom resource where you will have to replace `myonprembaw.mycompany.com:9443` with the actual hostname and port to the REST API of your traditional IBM Business Automation Workflow system: 
```
apiVersion: oidc.security.ibm.com/v1
kind: Client
metadata:
  name: onprem-client
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
- on Openshift:
   ```
   oc get client onprem-client -o=yaml
   ```
- on Kubernetes CNCF:
   ```
   kubectl get client onprem-client -o=yaml
   ```

The `status` section should mention that the request was successful:
```
status:
  conditions:
  - lastTransitionTime: "2025-06-07T12:09:11Z"
    message: OIDC client registration successful
    reason: CreateClientSuccessful
    status: "True"
    type: Ready
```

You can now inspect the secret named `onprem-baw-admin-client-secret`, which contains the following 2 keys that are needed in the remaining procedure: `CLIENT_ID` and `CLIENT_SECRET`.

<br/>

#### 1.1.6- Configuring the TAI to authenticate incoming queries

You can now create the TAI with 2 providers:
- `provider_1` to authenticate against the Cloud Pak Platform UI (Zen) requests from Workplace or IBM Process Federation Server that come with an `Authorization` HTTP header that contains a 'Bearer' token
- `provider_2` to authenticate against the Identity and Management (IM) service, by following the OAuth flow, requests from Workplace that do not come with an `Authorization` HTTP header that contains a 'Bearer' token

> **Note** : If SSO is already configured in some way (SpNego, SAML). you can skip all 'provider_2' properties 

1. Log in to the administrative console of the WebSphere Application server running traditional IBM Business Automation Workflow. 
2. Expand `Security` and click `Global Security`.
3. In `User account repository`, make sure that your server is set up to use the same user repository as your deployment on Red Hat OpenShift, which is a prerequisite to enable federation of the system by the IBM Process Federation Server that is deployed on Red Hat OpenShift.
4. In `Authentication`, click `Web and SIP security` and then click `Trust association`.
5. Under `Additional Properties`, click `Interceptors`.
6. Click `New` to add a `com.ibm.ws.security.oidc.client.RelyingParty` interceptor with the following custom properties in order to intercept all incoming queries coming from the IBM Process Federation Server that is deployed on Red Hat OpenShift.

|Name|Value|Description|
|---|---|---|
|`provider_1.identifier`|`cp4ba_zen`| You can choose another identifier if desired.|
|`provider_1.filter`|`Authorization%=Bearer;X-PFS-ID~=^.*\|\|Authorization%=Bearer;Referer%=https://cpd-myworkspace.apps.myocp.cp.mycompany.com` | Filter only requests with an `Authorization` HTTP header that contains a 'Bearer' token and that have either an `X-PFS-ID` HTTP header (request from IBM Process Federation Server) or a `Referer` HTTP header that is set on a URL from Workplace ([Zen URL](#1121--retrieving-the-zen-url)) |
|`provider_1.jwkEndpointUrl`|Location of the [cpd route](#1121--retrieving-the-zen-url), followed by `/auth/jwks` e.g.: `https://cpd-myworkspace.apps.myocp.cp.mycompany.com/auth/jwks`|Specifies the URL of the OIDC Provider's JSON Web Key (JWK) set document that contains the signing key required to validate the JSON Web Token (JWT) found in the `Authorization` HTTP header|
|`provider_1.useJwtFromRequest`|`required`| This provider requires a JWT for authentication. An OpenID Connect provider is not used.|
|`provider_1.issuerIdentifier`|`KNOXSSO`| Value expected for `iss` claim in the JWT issued by Zen|
|`provider_1.audiences`|`DSX`|Value expected for `aud` claim in the JWT issued by Zen|
|`provider_1.mapIdentityToRegistryUser`|`true`|The OpenID Connect RP maps the OpenID Connect authenticated user to the same user (by shortname) in the WebSphere Application Server user registry
|`provider_2.identifier`|`cp4ba_im_client` | You can choose another identifier if desired, but if doing so, you must adapt the `redirect_uris` property in the previous section|
|`provider_2.filter`|`Authorization!=Bearer;Referer%=https://cpd-myworkspace.apps.myocp.cp.mycompany.com\|\|Authorization!=Bearer;Referer%=https://myonprembaw.mycompany.com:9443` | Filter only requests without an `Authorization` HTTP header that contains a 'Bearer' token and with a `Referer` HTTP header which is either set on a URL from Workplace ([Zen URL](#1121--retrieving-the-zen-url)) or on a URL from the traditional IBM Business Automation Workflow system |
|`provider_2.clientId`|Value of the `CLIENT_ID` key of the `onprem-baw-admin-client-secret` secret (see previous section)|ID that is used to identify the OpenID Connect RP instance to the OpenID connect Provider server|
|`provider_2.clientSecret`|Value of the `CLIENT_SECRET` key of the `onprem-baw-admin-client-secret` secret (see previous section)|Specifies the secret that is used by the OpenID Connect Provider to secure messages that are sent to this RP client in callback requests|
|`provider_2.discoveryEndpointUrl`|Location of the [cp-console route](#1122--retrieving-the-identity-and-management-service-url), followed by `/oidc/endpoint/OP/.well-known/openid-configuration` e.g.: `https://cp-console-myworkspace.apps.myocp.cp.mycompany.com/oidc/endpoint/OP/.well-known/openid-configuration`|Specifies the OpenID Connect Provider's discovery endpoint URL.|
|`provider_2.useRealm`|Set this property to the default WebSphere realm name, e.g.: `defaultWIMFileBasedRealm` | Specifies the realm name to be used for each request to this provider| 
|`provider_2.nonceEnabled`|`true`|When this property is set to true, a nonce parameter is sent to the OpenID Connect provider on the authentication request|  
|`provider_2.createSession`|`true`|Set this property to true if you want the runtime to create a new HTTP session for each client request. The property is required when you run in a cluster environment and you need the JSESSIONID cookie to maintain session affinity.|

<br/>

> **Note:**: The documentation about the OIDC RP custom properties is in [OpenID Connect Relying Party custom properties](https://www.ibm.com/docs/en/was-nd/8.5.5?topic=party-openid-connect-relying-custom-properties). The `provider_1.filter` and `provider_2.filter` properties can be fine-tuned if required. You may leverage:
> - the `Referer` HTTP Header that is present in all queries from Workplace or IM
> - the `X-PFS-ID` HTTP Header that is present in all queries issued by IBM Process Federation Server and is set to `<namespace>-<ProcessFederationServer-custom-resource-name>`.
> The documentation about the `filter` property syntax is in [SAML web single sign-on (SSO) trust association interceptor (TAI) custom properties](https://www.ibm.com/docs/en/was/8.5.5?topic=swss-saml-web-single-sign-sso-trust-association-interceptor-tai-custom-properties)

<br/>

#### 1.1.7- Give the OIDC TAI a higher priority than the PreemptiveBasicAuthTAI

In a default BAW installation, 2 pre-existing TAIs are already configured:
- `com.ibm.bpm.security.PreemptiveBasicAuthTAI`
- `com.ibm.portal.auth.tai.HTTPBasicAuthTAI`

Upon incoming requests, it is required that the new OIDC TAI that you have configured is challenged before the `PreemptiveBasicAuthTAI`. 

As the TAIs are processed in the chronological order of their creation, you must delete the `PreemptiveBasicAuthTAI` TAI and recreate it. Here is the procedure to recreate the `com.ibm.bpm.security.PreemptiveBasicAuthTAI` TAI:
- in Webpshere console, go to `Security` > `Global security`
- expand `Web and SIP security` and click `Trust association`
- click on `Interceptors`
- click on `com.ibm.bpm.security.PreemptiveBasicAuthTAI`, and note the list of custom properties
- get back to the list of interceptors by clicking on `Interceptors`
- select the `com.ibm.bpm.security.PreemptiveBasicAuthTAI` interceptor and click `Delete`
- click `New...` 
- enter in `Interceptor class name` : `com.ibm.bpm.security.PreemptiveBasicAuthTAI`
- reset the properties exactly as they were initially defined. Any property which had a null value out of the box do not need to be recreated
- click `OK`

<br/>

Once the interceptor configuration is applied and saved, restart the IBM Business Automation Workflow server to activate the interceptor.

To check if the ordering of TAIs is as expected, you can enable the detailed trace specfication `*=info:com.ibm.ws.security.oidc.*=all:com.ibm.ws.security.openidconnect.*=all:com.ibm.ws.security.openid20.*=all:com.ibm.ws.security.web.*=all`, and in the traces.log file, upon each request to work on a task or start a new process instance, you should observe that the OIDC TAI is checked before the  PreemptiveBasicAuthTAI. For example:
```
[6/4/25 6:57:11:310 PDT] 000001ad TrustAssociat 3   Check if target interceptor [0]: com.ibm.portal.auth.tai.HTTPBasicAuthTAI ...
[...]
[6/4/25 6:57:11:310 PDT] 000001ad TrustAssociat 3   Check if target interceptor [1]: Jazz Security Architecture OIDC TrustAssociationInterceptor ...
[...]
[6/4/25 6:57:11:310 PDT] 000001ad TrustAssociat 3   Check if target interceptor [2]: com.ibm.bpm.security.PreemptiveBasicAuthTAI ...
```

If you have other TAIs registered in tWAS, you must similarly ensure that the filtering of each provider and the order of TAIs checks is not leading to an inappropriate TAI to be selected to process the request to work on a task or start a process instance from Workplace or Process Portal. If this occur, this would lead to HTTP 401 error.

#### 1.1.8- Expected authentication flows

##### 1.1.8.1- For requests with an Authorization Bearer token

Some requests from IBM Process Federation Server or from the federated Workplace and Process Portal will contain an Authorization Bearer JSON Web Token issued by the Cloud Pak Platform UI (Zen). 

You can track these requests in your Web Browser developer tools, in the Network tab. 

When such request is received by your traditional BAW, you must ensure that the OIDC TAI provider named `cp4ba_zen` is selected to process the request. You can confirm this by enabling the detailed trace specification `*=info:com.ibm.ws.security.oidc.*=all:com.ibm.ws.security.openidconnect.*=all:com.ibm.ws.security.openid20.*=all:com.ibm.ws.security.web.*=all` and you should see in the detailed traces the following lines (here for a call to https://myonprembaw.mycompany.com:9443/rest/bpm/wle/v1/systems REST API issued by IBM Process Federation Server to the traditional BAW federated system).

```
[6/4/25 6:50:29:642 PDT] 00000192 RelyingParty  >  OIDC: isTargetInterceptor(https://myonprembaw.mycompany.com:9443/rest/bpm/wle/v1/systems) Entry
[...]
[6/4/25 6:50:29:642 PDT] 00000192 CommonHTTPHea 3   Configured filter [Authorization%=Bearer;X-PFS-ID~=^.*||Authorization%=Bearer;Referer%=cpd-myworkspace.apps.myocp.cp.mycompany.com] [...]
[...]
[6/4/25 6:50:29:643 PDT] 00000192 CommonHTTPHea <  isAccepted returns[true] Exit
[6/4/25 6:50:29:643 PDT] 00000192 OidcTAIConfig <  getRelyingPartyConfig returns [com.ibm.ws.security.oidc.client.RelyingPartyConfig(index=[0], providerId=[cp4ba_zen], [...]
[...]
[6/4/25 6:50:29:643 PDT] 00000192 RelyingParty  3   The URL: [https://myonprembaw.mycompany.com:9443/rest/bpm/wle/v1/systems] accepted by OIDC RelyingParty
```

If you don't get these traces and that the request returns HTTP 401, then you must verify that:
- the different TAIs are processed in a order which does not discard the OIDC TAI to be challenged to process the incoming request.
- if you have other OIDC TAI providers defined, then you need to adjust the filters of each to ensure that the provider `cp4ba_zen`is the one selected upon such requests.


##### 1.1.8.2- For requests without Authorization Bearer token

When you work on task or attempt to start a new process instance from the federated Workplace or Process Portal, an iframe will be attempting to render a URL to the traditional BAW federated system. Such requests do not contain any Authorization header, and the authentication relies on a cookies set by Identity and Management (IM) service when logging in.

The expected flow is as follows:
- The user accesses the federated Workplace or Process Portal exposed on the CPD URL. ex: https://cpd-myworkspace.apps.myocp.cp.mycompany.com/baw-on-container/Workplace .
- If not yet authenticated, the user will be redirected to a login page hosted by the Identity and Management (IM) service exposed on CP URL. ex: https://cp-console-myworkspace.apps.myocp.cp.mycompany.com/oidc/login.jsp and he will enter his credentials.
- After this successful login a POST request to https://cp-console-myworkspace.apps.myocp.cp.mycompany.com/idprovider/v3/auth/j_security_check will:
  - set some cookies named like `WAS_x0123456789` and `WASReqURL` in the browser for future requests to URLs starting with https://cp-console-myworkspace.apps.myocp.cp.mycompany.com.
  - return HTTP 302 to redirect to the federated Workplace or Process Portal URL
- Then the federated Workplace or Process Portal shows up as expected.
- If the requests containing an Authorization Bearer Token are successfully processed, then you should see the list of processes and tasks showing up. In case of issue read [1.1.8.1- For requests with an Authorization Bearer token](#1181--for-requests-with-an-authorization-bearer-token) to troubleshoot.
- Then, while using the federated Workplace or Process Portal, when working on a task or starting a new process intance, some iFrames will try to access URLs hosted on the traditional BAW server. ex: https://myonprembaw.mycompany.com:9443/teamworks/process.lsw
- The com.ibm.ws.security.oidc.client.RelyingParty TAI provider named `cp4ba_im_client` must be selected and the request to https://myonprembaw.mycompany.com:9443/teamworks/process.lsw will:
  - return HTTP 401 with a javascript redirection to https://cp-console-myworkspace.apps.myocp.cp.mycompany.com/oidc/endpoint/OP/authorize . To ensure that the `cp4ba_im_client` OIDC TAI provider is properly selected to process the request, you can follow a similar procedure as the documented in [1.1.8.1- For requests with an Authorization Bearer token](#1181--for-requests-with-an-authorization-bearer-token)
  - using a `Set-Cookie` HTTP header, it will set a cookie named `OIDCSTATE_cp4ba_im_client`
  - execute a script to set another cookie named `OIDCREQURL_cp4ba_im_client`
- When the web browser accesses the https://cp-console-myworkspace.apps.myocp.cp.mycompany.com/oidc/endpoint/OP/authorize URL, the `WAS_x0123456789` and `WASReqURL` cookies set when logging in must be part of the request, and the IM service should successfully authenticating the user and then redirect to https://myonprembaw.mycompany.com:9443/oidcclient/cp4ba_im_client . _Note: If the cookies are missing, then the request to https://cp-console-myworkspace.apps.myocp.cp.mycompany.com/oidc/endpoint/OP/authorize will fail and redirection to https://cp-console-myworkspace.apps.myocp.cp.mycompany.com/oidc/login.jsp will happen again. To solve this you must identify why your web browser is not forwarding cookies as expected._
- This request to https://myonprembaw.mycompany.com:9443/oidcclient/cp4ba_im_client must contain the `OIDCSTATE_cp4ba_im_client` and `OIDCREQURL_cp4ba_im_client` cookies. It will then redirect back to the initially requested URL: https://myonprembaw.mycompany.com:9443/teamworks/process.lsw , and the new process instance will be succesfully started this time.


<br/>
<br/>

### 1.2- Enabling indexing on the traditional IBM Business Automation Workflow system

IBM Process Federation Server enables users to see a consolidated list of tasks, process instances and case instances from all IBM Business Automation Workflow systems in the federated environment. It is required that the tasks, process instances and case instances from the federated systems are all indexed into the same Federated Data Repository (Elasticsearch or Opensearch). Depending on the kind of task and the version of IBM Business Automation Workflow that you are federating, the procedure to have the data indexed into the Federated Data Repository is different.

To follow the procedure to enable the Federated Data Repository process indexing, you need to know the FDR credentials, FDR URL and exposed TLS certificate.

If you have not brought your own FDR when configuring IBM Process Federation Server, then an Opensearch installation has been deployed for you:
- to retrieve the host and port that your traditional IBM Business Automation Workflow system will use to access the Opensearch API:

  - If you are using an IBM Process Federation Server instance deployed on Openshift, run the following command:
    ```
    oc get route opensearch-route
    ```

  - If you are using an IBM Process Federation Server instance deployed on a Kubernetes CNCF platform which is not Openshift, run the following command:
    ```
    kubectl get ingress opensearch-ingress
    ```
- You will also need the credentials that your traditional Business Automation Workflow system will use to connect to Opensearch. These credentials can be found in a secret named `opensearch-admin-user`. This secret contains a single key which is the user name, and the value attached to this key is the password associated with this user.

<br/>

#### 1.2.1- Enabling the process indexing

##### 1.2.1.1- IBM Business Automation Workflow 24.0.0.0 and newer

Since 24.0.0.0, IBM Business Automation Workflow can directly index the tasks and process instances in the Federated Data Repository (FDR). To achieve this, you must follow procedure documented in [Enabling the Federated Data Repository process indexing](https://www.ibm.com/docs/baw/25.0.0?topic=indexes-enabling-federated-data-repository-process-indexing).

##### 1.2.1.2- IBM Business Automation Workflow 23.0.2 and older

 When federating IBM Business Automation Workflow 23.0.2 or older systems, the tasks and process instances must be indexed into the Federated Data Repository (FDR) by IBM Process Federation Server. For IBM Process Federation Server to index data, you must first enable indexing on the IBM Business Automation Workflow system, as described in [Enabling indexing on a federated system](https://www.ibm.com/docs/baw/25.0.0?topic=systems-enabling-indexing-federated-system).

As part of this configuration:
- you must create the change log tables in the Process Server database of the IBM Business Automation Workflow system. In the pod that runs IBM Process Federation Server, the database scripts are located in `/opt/ibm/wlp/ibmProcessFederationServer/wlp-ext/dbscripts/`. To retrieve these scripts, execute the following command:
  ```
  oc cp <IBM Process Federation Server_pod_name>:/opt/ibm/wlp/ibmProcessFederationServer/wlp-ext/dbscripts/ .
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

#### 1.2.2- Indexing case instances

To federate a Case management system, you must enable the indexing of case instances as documented here: [Indexing case instances](https://www.ibm.com/docs/baw/25.0.0?topic=indexes-indexing-case-instances).

<br/>
<br/>

### 1.3- Declaring Workplace as an allowed origin

<br/>

#### 1.3.1- Configure BAW for allowed origins

To allow Workplace to send queries to the traditional IBM Business Automation Workflow system REST API, Workplace URL must be declared as an allowed origin on the traditional system configuration. To achieve this, merge the following configuration in the same `100Custom.xml` file(s) as in the previous section:
```
<properties> 
    <server merge="mergeChildren">
        <rest merge="mergeChildren">
            <allowed-origins>https://cpd-myworkspace.apps.myocp.cp.mycompany.com</allowed-origins>
        </rest>
    </server>
</properties> 
```

> Note: After manually editing the `100Custom.xml` files, you must trigger a full resynchronization of the nodes: 
> 1. In the traditional WebSphere Application Server administrative console, click **System administration > Nodes**.
> 2. Select all the nodes and click **Full Resynchronize**.
> 3. Go to **Servers > Server Types > WebSphere application servers**.
> 4. Select the BAW server and click **Restart**.

<br/>

#### 1.3.2- Configure any frontend IHS loadbalancer for allowed origins

If the traditional IBM Business Automation Workflow system is configured to use a frontend IBM HTTP Server ( IHS ) loadbalancer ( including the WebSphere WebServer Plugin ) then IHS needs to be configured for allowed origins as well.

Update the configuration file on the HTTP server, for example httpd.conf or apache.conf, or the .htaccess file with the following settings.

1. **Do not** use the asterisk (`*`) wildcard character in any of the statements.

   ```plaintext-ibm
   Header always set Access-Control-Allow-Origin "https://cpd-myworkspace.apps.myocp.cp.mycompany.com"
   Header set Access-Control-Allow-Credentials "true"
   Header set Access-Control-Allow-Headers "DNT,X-CustomHeader,Keep-Alive,User-Agent,X-Requested-With,If-Modified-Since,Cache-Control,Content-Type,Authorization"
   Header set Access-Control-Allow-Methods "GET,POST,PUT,DELETE,OPTIONS"
   Header onsuccess unset Access-Control-Allow-Origin
   SetEnvIfNoCase REQUEST_METHOD OPTIONS skipwas=1
   ```

2. Allow the HTTP server to modify the request and response headers by enabling `mod_headers` in the configuration file.

3. Restart the HTTP server for the changes to take effect.

<br/>
<br/>

### 1.4- Configuring the Content Security Policy on the traditional IBM Business Automation Workflow system

The Content Security Policy must be properly configured to allow Workplace to work on tasks and process instances from the traditional system. 

For Workplace to render the Client Side Human Services without hitting violations of the Content Security Policies, you must edit the `ICP4ACluster` custom resource as follows:
1. In the `ICP4ACluster` custom resource, if there is an item in `spec.baw_configuration` where you have `host_federated_portal: true`.
2. In this item, merge the following properties:
   ```
       host_federated_portal: true
       federated_portal:
         content_security_policy_additional_origins:
           - 'https://myonprembaw.mycompany.com:9443'
   ```

The traditional IBM Business Automation Workflow system Content Security Policy must also be set appropriately. The traditional IBM Business Automation Workflow system URL must be declared as a valid source for the following directives:
 - `default-src`
 - `script-src`
 - `frame-src`
 - `frame-ancestors`
 - `object-src`
 - `connect-src`
 - `img-src`
 - `style-src`
 - `font-src`

The Workplace URL must also be declared in the `frame-ancestors` directive.

The following example shows how to set the traditional IBM Business Automation Workflow system Content Security Policy as appropriate:
```
# ./wsadmin.sh -conntype SOAP -port 8879 -host localhost -user bpmadmin -password Passw0rd -lang jython
WASX7209I: Connected to process "dmgr" on node Dmgr01 using SOAP connector;  The type of process is: DeploymentManager
WASX7031I: For help, enter: "print Help.help()"

wsadmin>print AdminTask.setBPMProperty(['-de', 'DE1', '-name', 'Security.ContentSecurityPolicyHeaderValue', '-value', "default-src data: blob: filesystem: about: ws: wss: 'unsafe-inline' 'unsafe-eval' https://myonprembaw.mycompany.com:9443 ; script-src 'unsafe-inline' 'unsafe-eval' https://myonprembaw.mycompany.com:9443 ; frame-src https://myonprembaw.mycompany.com:9443 ; object-src  https://myonprembaw.mycompany.com:9443 ; connect-src 'unsafe-inline' https://myonprembaw.mycompany.com:9443 ; img-src data: blob: 'unsafe-inline' https://myonprembaw.mycompany.com:9443 ; style-src data: blob: 'unsafe-inline' https://myonprembaw.mycompany.com:9443 ; font-src data: blob: 'unsafe-inline' https://myonprembaw.mycompany.com:9443 ; frame-ancestors https://myonprembaw.mycompany.com:9443 https://cpd-myworkspace.apps.myocp.cp.mycompany.com"])
true
wsadmin>AdminConfig.save()
''
```


> **Note:** The documentation about the Security-hardening properties is [here](https://www.ibm.com/docs/en/baw/24.x?topic=environment-security-hardening-properties).

<br/>

---

## 2- Configuring IBM Process Federation Server to federate the traditional IBM Business Automation Workflow system

### 2.1- Planning your federation

Now that the IBM Business Automation Workflow system is properly configured to be federated, you have options regarding how data will be indexed from the traditional IBM Business Automation Workflow system relational database to the Federated Data Repository (Elasticsearch/Opensearch). These options depend on the version of IBM Business Automation Workflow system.

Whichever option, you also have to ensure that your OCP/Kubernetes cluster is configured to allow outbound communication from the IBM Process Federation Server pods to the on-prem IBM Business Automation Workflow. For more details, see the IBM Documentation section about [Configuring cluster security](https://www.ibm.com/docs/cloud-paks/cp-biz-automation/25.0.0?topic=security-configuring-cluster).

#### 2.1.1- IBM Business Automation Workflow 24.0.0.0 and newer

Since 24.0.0.0, IBM Business Automation Workflow can directly index the tasks, process instances and case instances in the Federated Data Repository (FDR). If you followed the procedures from [1.2.1.1 - Enabling process indexing on the traditional IBM Business Automation Workflow 24.0.0.0 and newer](#1211--ibm-business-automation-workflow-24000-and-newer) and/or [1.2.2. Indexing case instances](#122--indexing-case-instances), then the IBM Business Automation Workflow system is already indexing data into the FDR as depicted in the following diagram: 

![Indexing options diagram for 24.0.0.0 and newer](/documentation/images/indexing-24000.png)

You can now proceed to the following section: [2.2- Creating a FederatedSystem custom resource to reference your traditional IBM Business Automation Workflow system](#22--creating-a-federatedsystem-custom-resource-to-reference-your-traditional-ibm-business-automation-workflow-system).

<br/>

#### 2.1.2- IBM Business Automation Workflow 23.0.2 and older

 If you followed the procedure from [1.2.1.2 - Enabling process indexing on the traditional IBM Business Automation Workflow 23.0.2 and older](#1212--ibm-business-automation-workflow-2302-and-older), then the IBM Business Automation Workflow process engine is properly configured to be federated. You now have two options regarding how data will be indexed from the traditional IBM Business Automation Workflow system relational database to the Federated Data Repository (Elasticsearch/Opensearch):
- Configure the IBM Process Federation Server running on Openshift/Kubernetes to declare the federated system, a retriever, and also an indexer to index data from the traditional IBM Business Automation Workflow system database to the federated system into the Federated Data Repository already used by IBM Process Federation Server running on Openshift/Kubernetes.
- Configure a traditional IBM Process Federation Server deployment that will be colocated with your traditional IBM Business Automation Workflow system database and that will perform indexing from the traditional IBM Business Automation Workflow process engine database to the Federated Data Repository already used by IBM Process Federation Server running on Openshift/Kubernetes. With this approach, the IBM Process Federation Server running on Openshift/Kubernetes is not responsible for indexing data from the traditional IBM Business Automation Workflow federated system and will only declare the federated system and a retriever to the traditional IBM Business Automation Workflow federated system. This approach provides better performance if there is a network latency larger than 10ms between the traditional IBM Business Automation Workflow system site and IBM Process Federation Server running on Openshift/Kubernetes.

![Indexing options diagram for 23.0.2 and older](/documentation/images/indexing-2302.png)

<br/>
<br/>

### 2.2- Creating a FederatedSystem custom resource to reference your traditional IBM Business Automation Workflow system

From version 23.0.1, the IBM Process Federation Server operator uses a new custom resource Definition named "**FederatedSystem**" to seamlessly federate IBM Business Automation Workflow systems. For each system to federate, an instance of **FederatedSystem** custom resource must be instantiated. This instantiation of **FederatedSystem** custom resource is performed internally by the operators when federating systems that are deployed on your Openhift/Kubernetes cluster. But to federate a traditional IBM Business Automation Workflow system, you must manually create a **FederatedSystem** custom resource.

You can choose between the following options when configuring the **FederatedSystem** custom resource for your traditional IBM Business Automation Workflow federated system:
- [Basic configuration](#221--basic-configuration): provide a minimum set of settings, and let the IBM Process Federation Server operator generate the [Liberty configuration dropin](https://www.ibm.com/docs/en/was-liberty/base?topic=files-using-configuration-dropins-folder-specify-server-configuration) to federate the traditional IBM Business Automation Workflow system.
- [Advanced configuration](#222--advanced-configuration): provide your own [Liberty configuration dropin](https://www.ibm.com/docs/en/was-liberty/base?topic=files-using-configuration-dropins-folder-specify-server-configuration) that configures IBM Process Federation Server to federate the traditional IBM Business Automation Workflow system.
- [Mixing basic and advanced configurations](#223--mixing-basic-and-advanced-configurations): use [basic federation](#221--basic-configuration) to provide a minimum set of settings and let the IBM Process Federation Server operator generate the [Liberty configuration dropin](https://www.ibm.com/docs/en/was-liberty/base?topic=files-using-configuration-dropins-folder-specify-server-configuration), and use [advanced configuration](#222--advanced-configuration) to provide your own [Liberty configuration dropin](https://www.ibm.com/docs/en/was-liberty/base?topic=files-using-configuration-dropins-folder-specify-server-configuration) that overrides or augments the generated configuration.

<br/>

#### 2.2.1- Basic configuration

The basic configuration is the most straightforward approach: you provide the base URL to the traditional IBM Business Automation Workflow system, the corresponding credentials and TLS certificate, and the operator takes care of retrieving all required data from the system to configure IBM Process Federation Server to federate the traditional IBM Business Automation Workflow federated system. 

If you also want IBM Process Federation Server to index data from the traditional IBM Business Automation Workflow process engine database to the Federated Data Repository, some additional properties about the traditional IBM Business Automation Workflow system database are also required to [enable indexing](#2214--enabling-indexing).

##### 2.2.1.1- Creating a secret with credentials to the federated system

When using the basic configuration, the IBM Process Federation Server operator needs to perform a query against the `/rest/bpm/wle/v1/systems` REST API exposed by the traditional IBM Business Automation Workflow system. The credentials used to authenticate this request are sensitive information and must be packaged in a secret. This secret must contain the following keys: `serverUser` and `serverPassword`.

For example:
```sh
kubectl create secret generic onprem-credential --from-literal=serverUser=user --from-literal=serverPassword=password
```

<br/>

##### 2.2.1.2- Creating a secret with the TLS certificate

You must also create a Kubernetes secret that contains the traditional IBM Business Automation Workflow system root certificate that will later be added to the IBM Process Federation Server truststore.

To get the traditional IBM Business Automation Workflow system root certificate, follow these steps:

1. In the traditional WebSphere Application Server administrative console, click **Security > SSL certificate and key management > Key stores and certificates > CellDefaultTrustStore > Signer certificates**.
2. Select the root certificate and click **Extract**.
3. Name the file. For example, name the file `on-prem-cert.crt`.
4. For the data type, select Base64-encoded ASCII data.
5. Click **Apply**. From the message, note where the certificate is stored on the file system. Then transfer it to the cloud platform where IBM Process Federation Server is deployed.
6. You can create the Kubernetes secret as shown in the following example:
   ```
   kubectl create secret generic onprem-cert-secret --from-file=tls.crt=/path/to/on-prem-cert.crt
   ```

<br/>

##### 2.2.1.3- Creating the FederatedSystem custom resource for basic configuration
You can now create the **FederatedSystem** custom resource.

Here is an example of a **FederatedSystem** custom resource that does not declare an indexer, assuming that indexing is performed by an IBM Business Automation Workflow 24.0.0.0 and newer system running on premise that indexes in the same Elasticsearch cluster as the one used by the IBM Process Federation Server running on Openshift/Kubernetes:

```
cat << EOF |  oc apply -f -
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
EOF
```

The complete documentation of the **FederatedSystem** custom resource properties can be found in [Federated system parameters](https://www.ibm.com/docs/cloud-paks/cp-biz-automation/25.0.0?topic=deployment-federated-system-parameters).

<br/>

##### 2.2.1.4- Enabling indexing

> **Note: this procedure is not applicable when federating Business Automation Worklow 24.0.0.0 and newer. It may only apply to version 23.0.2 and older depending on the [option](#212--ibm-business-automation-workflow-2302-and-older) you chose.**

If you plan to also have IBM Process Federation Server running on Openshift/Kubernetes to perform the indexing of the traditional IBM Business Automation Workflow tasks and process instances into the Federated Data Repository, you must add a section in the **FederatedSystem** custom resource specification to provide details related to the database used by the traditional IBM Business Automation Workflow system.

The supported indexer datasources are:
- DB2
- PostgreSQL
- Oracle
- Microsoft SQL Server

Depending on the database used, you must provide the corresponding section. For example, for a DB2 database, you must add a section `spec.db2Indexer` as documented in [Federated system parameters](https://www.ibm.com/docs/cloud-paks/cp-biz-automation/25.0.0?topic=deployment-federated-system-parameters).

When the database is accessed with an SSL connection, you must also package in a secret the TLS public key certificate to add to the IBM Process Federation Server truststore for the SSL connection to the database to be authorized, similarly to what is documented [here](#2212--creating-a-secret-with-the-tls-certificate). Then, reference this secret in the **FederatedSystem** custom resource specification. For example, for a DB2 indexer, the certificate would have to be referenced under the key:  `spec.db2Indexer.sslCertSecret`.


<br/>
<br/>

#### 2.2.2- Advanced configuration

##### 2.2.2.1- Creating a secret with the Liberty configuration dropin
Instead of using the basic federation, you can use the advanced federation where you will have the responsibilty for creating a secret that holds keys to be mounted in the `/configDropins/overrides` folder of the pod running IBM Process Federation Server.

You must create an entry in this secret that contains a key that ends with ".xml". This entry contains the following configuration elements:

* To federate the IBM Business Automation Workflow process engine:
  * the declaration of the `<ibmPfs_federatedSystem>` element
  * the datasource of the traditional IBM Business Automation Workflow system (`<datasource>` element). This is required if indexing is to be performed from IBM Process Federation Server running on Openshift/Kubernetes.
  * the `<ibmPfs_bpdRetriever>` element
  * the `<ibmPfs_bpdIndexer>` element if indexing is to be performed from IBM Process Federation Server running on Openshift/Kubernetes

* to federate the IBM Business Automation Workflow Case management system:
  * the declaration of the `<ibmPfs_federatedSystem>` element
  * the `<ibmPfs_caseRetriever>` element


You can refer to [Configuration properties for federated systems](https://www.ibm.com/docs/baw/25.0.0?topic=systems-configuration-properties-federated) for detailed information about the above configuration elements.

Note that instead of creating this configuration from scratch, the procedure of [mixing basic and advanced configurations](#223--mixing-basic-and-advanced-configurations) may be simpler and less error-prone.
<br/>

> **Note:**
> IBM Process Federation Server containers has a set of predefined libraries that you can reference when you declare the jdbcDriver used by your datasource:
> * For a Db2 datasource, use `<jdbcDriver libraryRef="DB2JCC4Lib"/>`
> * For a PostgreSQL datasource, use `<jdbcDriver libraryRef="postgresqlJDBCLib"/>`
> * For an Oracle datasource, use `<jdbcDriver libraryRef="OracleLib"/>`
> * For a Microsoft SQL Server datasource, use `<jdbcDriver libraryRef="MSJDBCLib"/>`

<br/>

##### 2.2.2.2- Creating the secrets to set up the truststore

To allow SSL connection to the traditional IBM Business Automation Workflow federated system, and to its relational database, you must ensure that TLS certificates are added to the IBM Process Federation Server truststore by packaging them into secrets similarly to what is documented [here](#2212--creating-a-secret-with-the-tls-certificate), and then reference this secret in `spec.certificates.secrets`.

<br/>

##### 2.2.2.3- Creating the FederatedSystem custom resource for advanced federation

Once this secret is created, you can create the **FederatedSystem** custom resource.

The following example shows a FederatedSystem custom resource that uses the advanced federation:

```
cat << EOF |  oc apply -f -
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
EOF
```

<br/>
<br/>

#### 2.2.3- Mixing basic and advanced configurations

It is possible to set up a **FederatedSystem** custom resource by following the [basic configuration](#221--basic-configuration) procedure - so that the IBM Process Federation Server operator generates the [Liberty configuration dropin](https://www.ibm.com/docs/en/was-liberty/base?topic=files-using-configuration-dropins-folder-specify-server-configuration) that will declare the federated system - and also provide a secret referenced as `spec.advancedConfig` as described in [advanced configuration](#222--advanced-configuration) to override the default configuration or add new configuration tags like `<ibmPfs_launchableEntity>` as documented [here](https://www.ibm.com/docs/baw/25.0.0?topic=portal-dashboards-processes-services).

A relevant use-case is when the same process applications are deployed on the traditional IBM Business Automation Workflow (where there are on-going instances of these process applications) and on the container-based Workflow server (IBM Business Automation Workflow or IBM Workflow Process Service) and that you want new instances of these process applications to be only created on the container-based Workflow server and not on the traditional IBM Business Automation Workflow system then you have to override the default `<ibmPfs_federatedSystem>` configuration of the traditional IBM Business Automation Workflow system to give it a lower priority:

1. the first action to take is to first follow the [basic configuration](#221--basic-configuration) procedure. 

2. once the system is federated with a default configuration you will search for the secret which contains the default configuration. This secret has the same name as the **FederatedSystem** custom resource. 

3. in this secret you will find a single key which contains the Liberty configuration dropin which declares the federated system. From the key value, you must copy-paste the `<ibmPfs_federatedSystem>` configuration tag and re-use it in a new secret but add `launchListPriority="1001"` property. Setting `launchListPriority` to 1001 is a higher value than the default `launchListPriority` of the other federated systems which have a default value of 1000. A lower value for `launchListPriority` means a higher priority. This way, if the same process app is deployed on the traditional IBM Business Automation Workflow system and on a Workflow server running on containers, and that both systems are available, then when a user will start a new instance of this process app, it will be always processed on the Workflow server running on containers. This secret can be created as follows:
   ```
   cat << EOF |  oc apply -f -
   kind: Secret
   apiVersion: v1
   metadata:
     name: onprem-baw-federatedsystem-adv
     namespace: demo-project
   type: Opaque
   stringData:
     zzz-on-prem-baw.xml: |
       <server>
        <ibmPfs_federatedSystem id="6b4d5b75-c1f0-4561-b85a-af8f299b6016" systemType="SYSTEM_TYPE_WLE"
           indexName="6b4d5b75-c1f0-4561-b85a-af8f299b6016"
           restUrlPrefix="https://onprem-host:9443/rest/bpm/wle" 
           taskCompletionUrlPrefix="https://onprem-host:9443/teamworks"
           allowedOrigins="*" 
           authenticationMechanism="IBM Process Federation Server_ACCESS_TOKEN"
           indexProcessInstances="true"
           launchListPriority="1001"
         />
       </server>
   EOF
   ```
   Notice that the key starts with "zzz". This is because Liberty processes the configuration dropins in alphabetical order and we must ensure that the file name of the configuration dropin is alphabetically greater than the file name of the default configuration dropin to override it.

4. Once the secret is created, you have to edit the **FederatedSystem** custom resource to reference the secret as `spec.advancedConfig`:
   ```
   cat << EOF |  oc apply -f -
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
     advancedConfig: onprem-baw-federatedsystem-adv
   EOF
   ```

If you do not want to override the existing configuration but only want to add new configuration elements to it, like `<ibmPfs_launchableEntity>` as documented [here](https://www.ibm.com/docs/baw/25.0.0?topic=portal-dashboards-processes-services), you can directly create a config dropin which contains the `<ibmPfs_launchableEntity>` configuration tags, package it as a secret, and create a **FederatedSystem** custom resource like in the [basic configuration](#221--basic-configuration) procedure, but with the additional `spec.advancedConfig` parameter to reference the secret containing the configuration dropin containing the `<ibmPfs_launchableEntity>` elements.

<br/>

---

## 3- Verifying that the traditional IBM Business Automation Workflow system is successfully federated

Once the **FederatedSystem** custom resource is created, you can verify that the system is successfully federated by calling the following IBM Process Federation Server API:
`/rest/bpm/federated/v1/system`.

In the response, verify that in the `federationResult` array, there is an item for your traditional IBM Business Automation Workflow federated system and that the `statusCode` value of this item is `"200"`.

For more details about how to access the IBM Process Federation Server REST API, see [this documentation](/documentation/PFS-Statefulset.md#accessing-the-rest-api).

<br/>

---

**Parent topic:** [Administering and operating IBM Process Federation Server](../README.md)

**Index:** [Documentation index](../README.md#documentation-index)
