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

<pre>
HTTP/1.1 302 Found
Location: {authentication_endpoint}?{parameters}
</pre>

As a result, the end-user client will issue the following request:

##### Request command

<pre>
GET {authentication_endpoint_path}?{parameters} HTTP/1.1
Host: {authentication_endpoint_host}
</pre>

Knowing that we consider the context of our [preproduction](#ref-2-1) environment, the authentication endpoint path is `/a/auth` under the `accounts.ozwillo-preprod.eu` host, according to the <a href="http://kernel.ozwillo-preprod.eu/.well-known/openid-configuration" target="_blank">Well-Known URI Registry</a>.

Before describing the query `{parameters}` are described in the following table, the request would then looks like:

<pre>
GET /a/auth?{parameters} HTTP/1.1
Host: accounts.ozwillo-preprod.eu
</pre>

##### Query parameters

| Field name | Field description | Field type and format |
| :-- | :-- | :-- |
| **response_type** | determines the authorization processing flow, in our case the value is always `code` | string |
| **client_id** | the `client_id` associated to the application instance | string |
| **scope** | list of scopes requested by the instance, it must at least contain `openid` | strings separated by spaces |
| **redirect_uri** | the redirection URI to which the response will be sent | URI string |
| **state** | opaque value used to maintain state between the request and the callback | string |
| **nonce** | unique random string used to mitigate replay attacks | string |

As a result, a request could look like:

<pre>
GET /a/auth?
 response_type=code
 &client_id={client_id}
 &scope=openid%20profile
 &redirect_uri=https://app.example.com/cb
 &state=security_token%3D{random_value}%26url%3Dhttps%3A%2F%2Fapp.example.com%2FmyHome
 &nonce={another_random_value} HTTP/1.1
Host: accounts.ozwillo-preprod.eu
</pre>

In this example, some characters (among space = & : /) have been URL-escaped.

#### #2 Ozwillo response
{: #ref-4-3-2}

Several operations are then conducted on Ozwillo side:

1. validate the incoming authentication request from step #1;
2. authenticate the user;
3. ask user to accept scopes claimed by the instance.

##### SUCCESS

If all of these succeed, and if the `redirect_uri` value specified in [step #1](#ref-4-3-1) matches one of the `redirect_uris` specified during the [provisioning acknowledgement](#ref-3-2-3), the following response will be sent:

<pre>
HTTP/1.1 302 Found
Location: https://app.example.com/cb?state=security_token%3D{random_value}%26url%3Dhttps%3A%2F%2Fapp.example.com%2FmyHome&code={another_random_value}
</pre>

This redirect workflow means your server will finally receive the following request sent from the end-user navigator:

<pre>
GET /cb?state=security_token%3D{random_value}%26url%3Dhttps%3A%2F%2Fapp.example.com%2FmyHome&code={another_random_value} HTTP/1.1
Host: app.example.com/
</pre>

##### FAILURE

The same `redirect_uri` callback will be notified of the authentication error according to the <a href="http://openid.net/specs/openid-connect-core-1_0.html#AuthError" target="_blank">spec</a>.

#### #3 Response validation
{: #ref-4-3-3}

You should verify that both the `security_token` (within the `state`) and the `nonce` are the ones you sent first to Ozwillo, to decide that you can trust the response.

TODO: usage of state
{: .todo}

#### #4 Requesting an access token
{: #ref-4-3-4}

#### #5 Access token validation
{: #ref-4-3-5}

### FAQ
{: #ref-4-4}