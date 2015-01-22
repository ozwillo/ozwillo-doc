## User Authentication
{: #ref-4} 

### Introduction
{: #ref-4-1}

Ozwillo implements the <a href="http://openid.net/connect/" target="_blank">OpenID Connect</a> protocol acting as an OpenID Provider, which is defined as (see OpenID <a href="http://openid.net/specs/openid-connect-core-1_0.html#Terminology" target="_blank">terminology</a>):

> **OAuth 2.0 Authorization Server** that is capable of Authenticating the End-User and providing Claims to a Relying Party about the Authentication event and the End-User.

In this respect, the provider acts as a Relying Party:

> **OAuth 2.0 Client** application requiring End-User Authentication and Claims from an OpenID Provider.

You may refer to the previous [Authentication in brief](#ref-1-5-2) and [scopes](#ref-1-5-3) paragraphs for a remainder, but the outcome of a successful authentication is the creation of an `access_token` that links three resources (during a limited period of time):

- a user
- an application instance identified by a `client_id`
- scopes

Then, anytime the application instance tries to interact with user-related resources (typically due to user action) through Ozwillo API, this `access_token` is sent to Ozwillo so that it can check the operation is allowed.

That's how Ozwillo provides both authentication *and* authorization features.

### Prerequesite
{: #ref-4-2}

You will [soon](#ref-4-3-1) see that you ask Ozwillo to authenticate users against a `client_id`. Since this in an [outcome](#ref-3-2-1) of the provisioning process, you should implement provisioning first.

In the case that what you offer is not an instantiable application, but a unique and singleton one, your product may be considered as a service, and you may ask for a `client_id` to <a mailto="providers@ozwillo.com">providers@ozwillo.com</a>.

### Authorization code flow
{: #ref-4-3}

You may refer to the official OpenID Connect <a href="http://openid.net/specs/openid-connect-core-1_0.html#CodeFlowSteps" target="_blank">authorization code flow</a> in addition to this documentation.

Two particularities may be noted in comparison with the official spec:

- if there is a one-to-one mapping between `client_id`s and application instances, in Ozwillo we also add a one-to-many relation between applications instances and services. It means that several services (end-user endpoints) may exist within the same `client_id`;

- the ID Token sent by Ozwillo as an outcome of [step #3](#ref-4-3-3) contains the <a href="http://openid.net/specs/openid-connect-core-1_0.html#IDToken" target="_blank">official</a> required fields, plus Ozwillo specific settings (`app_user` and `app_admin` boolean values).

Still within the context of the official spec, some optional features may not be implemented as of today, for instance passing a `max_age` parameter to the authentication endpoint.

#### #1 Authentication request
{: #ref-4-3-1}

In this step, the application instance has detected an authentication of the user is needed and delegates it to Ozwillo by **redirecting** the end-user request to Ozwillo authentication endpoint.

It means the end-user client will issue the following request:

##### Request command

<pre>
GET {authentication_endpoint}?{parameters} HTTP/1.1
Host: {authentication_host}
...
</pre>

Knowing that we consider the context of our [preproduction](#ref-2-1) environment, the authentication endpoint is `/a/auth` under the `accounts.ozwillo-preprod.eu`, according to the <a href="http://kernel.ozwillo-preprod.eu/.well-known/openid-configuration" target="_blank">Well-Known URI Registry</a>.

The query `{parameters}` are described in the following table.

##### Query parameters

| Field name | Field description | Field type and format |
| :-- | :-- | :-- |
| **response_type** | determines the authorization processing flow, in our case the value is always `code` | string |
| **client_id** | the `client_id` associated to the application instance | string |
| **scope** | list of scopes requested by the instance, it must at least contain `openid` | strings separated by spaces |
| **redirect_uri** | the redirection URI to which the response will be sent | URI string |
| **state** | opaque value used to maintain state between the request and the callback | string |
| **nonce** | unique random string used to mitigate replay attacks | string |

As a result, a possible complete request should look like:

<pre>
https://accounts.ozwillo-preprod.eu/a/auth?response_type=code&client_id={client_id}&scope=openid%20profile&redirect_uri=https://app.example.com/cb&state=security_token%3D{random_value}%26url%3Dhttps%3A%2F%2Fapp.example.com%2FmyHome&nonce={another_random_value}
</pre>

In this example, some characters (among space = & : /) have been URL escaped.

It's important to note that the redirect to `redirect_uri` will indeed be asked through Ozwillo response if its value matches one of the `redirect_uris` specified during the provisioning acknowledgement [step](#ref-3-2-3) regarding the Service object.

This redirect workflow means that the call to `redirect_uri` that concludes a succesful [step #2](#ref-4-3-2) is requested from the end-user navigator.

#### #2 Ozwillo processing
{: #ref-4-3-2}

### #3 Requesting an access token
{: #ref-4-3-3}

### #4 Validating the access token
{: #ref-4-3-4}

### FAQ
{: #ref-4-4}