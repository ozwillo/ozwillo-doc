## Provisioning
{: #ref-3}

This section explains the provisioning protocol triggered between Ozwillo and the provider APIs when a [purchase act](#def-purchase-act) occurs on the portal. The outcome of a successful provisioning is an application instance that is both:

1. allocated and reachable on the provider servers
2. declared with appropriate settings, especially regarding access rights, on Ozwillo

where (2) enables the creation of [desk shortcuts](#def-desk-shortcuts) so that users can access (1)
{: .where}

The creation of an application instance on the provider side may imply configuration changes or new resources allocation. These implementation details are left unconstrained: the protocol focuses only on API communication. It means existing deployment operations does not have to be affected and should rather be exposed through this protocol.

### Prerequesite
{: #ref-3-1}

Since provisioning starts when a user performs a purchase act, it means the purchased application is declared in the [catalog](#def-catalog) **and** Ozwillo knows how to forward the request to the provider through its [app factory](#def-app-factory).

Thus the following information is needed to have a well described *and* installable application in the catalog:

##### Commercial information
{: #ref-3-1--1}

| Field name | Field description | Field type and format |
| :-- | :-- | :-- |
| **name** | name in the default language (try to remain below 20 characters) | string |
| name#{lg} | name in the given language {lg} (try to remain below 20 characters) | string |
| **description** | description in the default language | markdown string |
| description#{lg} | description in the given language | markdown string |
| **tos_uri** | terms of service URI implicitly accepted on purchase, in the default language | URI string |
| tos_uri#{lg} | terms of service URI implicitly accepted on purchase, in the given language | URI string |
| **policy_uri** | privacy policy URI implicitly accepted on purchase, in the default language | URI string |
| policy_uri#{lg} | privacy policy URI implicitly accepted on purchase, in the given language | URI string |
| **icon** | app icon URI in the default language, a 64px x 64px png is expected | URI string |
| icon#{lg} | app icon URI in the given language, a 64px x 64px png is expected | URI string |
| screenshot_uris | list of screenshot URIs, a 850px x 450px png are expected | array of URI strings |
| **contacts** | list of URLs to contact the provider or its support | array of URI (URL or mailto) strings |

A few comments on this table:

- `{lg}` identifies a language following <a href="https://tools.ietf.org/html/bcp47" target="_blank">BCP 47</a>. Some examples regarding the fields introduced above: `name#es`, `description#fr_FR`, `description#fr_BE`.
- the *default* language information is what will be displayed if the user preferred language information is not available (no matching custom `{lg}` version). It can be whatever language suits best your target audience;
- <a href="http://daringfireball.net/projects/markdown/syntax" target="_blank">here</a> is a description of the markdown syntax. In particular, you may include raw text separated by two line breaks to shape paragraphs.

##### Store filters
{: #ref-3-1--2}

| Field name | Field description | Field type and format |
| :-- | :-- | :-- |
| **payment_option** | "FREE" or "PAID" setting | string |
| **target_audience** | is the application intended to "CITIZENS", "PUBLIC_BODIES" and/or "COMPANIES" | array of strings |
| category_ids | IDs of the application store categories, not supported for the moment | array of strings |
| visible | if false, the application is not visible in the application store | boolean |

##### App factory

| Field name | Field description | Field type and format |
| :-- | :-- | :-- |
| **instantiation_uri** | instantiation endpoint | URI string |
| **instantiation_secret** | secret used to compute the instantiation request signature | string |
| **cancellation_uri** | cancellation endpoint, to cancel pending instantiations | URI string |
| **cancellation_secret** | secret used to compute the cancellation request signature | string |

There is no official spec about secret strings, but 30 characters within a rich character set (richer than hexadecimal) can be seen as a bare minimum. Here is a <a href="https://www.grc.com/passwords.htm" target="_blank">tool</a> to generate random strings.

##### Summary

You will likely need to read the remainder of this section to fully understand how fields are used, especially those related to the app factory. It means these setup steps are required before receiving provisioning requests from Ozwillo:

1. gather all parameters info needed as described in the previous tables
2. send it to <a mailto="providers@ozwillo.com">providers@ozwillo.com</a> along with your display name as an application provider (typically the name of your company)
3. you will be notified when the application is made available on the [preproduction](#ref-2-1)

To ease the process, you may submit a first and simplified version of the commercial info fields, focusing on simple versions of **required** fields, and resubmit it later with enriched contents.

### Protocol
{: #ref-3-2}

#### #1 Ozwillo request
{: #ref-3-2-1}

##### Description

When a purchase act occurs, Ozwillo creates a new [application instance](def-application-instance) in its catalog, in a pending state (since it has not been provisioned for the moment). This instance is given a unique and constant `instance_id` and credentials (`client_id` and `client_secret`). The `client_secret` must remain to be known only by Ozwillo and the provider.

The actual behavior of Ozwillo is that `instance_id` and `client_id` are given the same unique GUID value, but this is really an implementation detail subject to change. The spec is that `instance_id` is an identifier that won't change over time, when `client_id` may be refreshed.
{: .focus .important}

##### Request command

Don't forget to read our [conventions](#ref-1-5) and [recommendations](#ref-2-4).
{: .focus .soft}

The following HTTP request is sent from Ozwillo to the provider at `instantiation_uri` (decomposed in `instantiation_path` and `instantiation_host` below). 

<pre>
POST {instantiation_path} HTTP/1.1
Host: {instantiation_host}
Accept: application/json, application/*+json
X-Hub-Signature: sha1={HMAC-SHA1 digest of payload}
Content-Type: application/json;charset=UTF-8
</pre>

The body of this request is encoded in JSON. A HMAC-SHA1 hash of the JSON string is computed using the application's `instantiated_secret` as key with an uppercase hexadecimal output format. This computed HMAC-SHA1 signature is then inserted in the X-Hub-Signature (as per <a href="https://pubsubhubbub.googlecode.com/git/pubsubhubbub-core-0.4.html#authednotify" target="_blank">PubSubHubbub Core 0.4</a>).
{: #ref-hmac-signature}

The provider must recompute the signature of the request body with the same secret key and method. If signatures match, this reasonably certifies the instantiation request comes from Ozwillo (which is the only one knowing the `instantiated_secret`) and then can be trusted. If not, you may log the request for further analysis, and ignore it.

##### Request body

| Field name | Field description | Type |
| :-- | :-- | :-- |
| **instance_id** | unique and constant identifier of the instance | string |
| **client_id** | used for authentication purposes | string |
| **client_secret** | used for authentication purposes | string |
| **user** | a description of [purchaser](#def-purchaser), containing at least its Ozwillo user id | User object |
| organization | a description of the organization, containing at least Ozwillo organization id and name | Organization object |
| **instance_registration_uri** | the endpoint that must be used to acknowledge the provisioning back to Ozwillo | URI string |
{: .request}

It's important to know that purchasers may install an application either on behalf an organization, either for their personal user. In the latter case, there is no organization associated to the purchase, so there is no `organization` field in the request body.

That said, if your application `target_audience` (as declared in [store filters](#ref-3-1--2)) does not contain `CITIZENS`, it means purchase acts will always be on behalf an organization and thus the `organization` field will be sent.

##### Response from provider

Accepted status codes:

- 2xx: request acknowledged, the app factory will do the job;
- 4xx: request cannot be honoured (for instance, the organization already owns an instance of this application and it is limited to one);
- any other non-2xx will be declared a failure too (maybe we'll follow redirect, but let's say for now that we won't).

#### #2 Provider provisioning
{: #ref-3-2-2}

By provider provisioning we mean the installation process needed on the provider servers. As previously explained in the [introduction](#ref-3) to this section, this step can be implemented in various ways:

- fully automatic (for instance - but not restricted to - a multitenant architecture where a single software instance manages several application instances);
- half automatic (requires a manual validation);
- fully manual (requires a manual installation).

Moreover this process also depends on existing software and architectural choices. That being said, we can give the following guidelines:

- there should be an internal process that stores [step #1](#ref-3-2-1) request properties and link them to services declared in [step #3](#ref-3-2-1). This configuration will be useful during authentication (for instance, you will soon see that a service accessible at a given `service_uri` has to know to what `instance_id` it belongs);
- you should be able to treat both preproduction and production provisioning requests, and identify them as such. Knowing you will start implementing provisioning against Ozwillo preproduction, and then move it to production while keeping the preproduction tests up, you need to know to what API [hosts](#ref-2-1) your servers communicate with.

From a user perspective, until the [step #3](#ref-3-2-3) is done, purchase acts are materialized as grey and disabled desktop shortcuts.

#### #3 Provider acknowledgement
{: #ref-3-2-3}

##### Description

In this step, the provider declares the precise configuration of the installed instance, including in particular:

- services composing the instance (the user endpoints);
- scopes needed for the instance to work (for instance the OpenID Connect `email` scope if you need to know the user email to configure the instance);
- scopes declared by the instance, so that other instances may require them as needed if they want to use this very instance exposed API.

This list is not exhaustive and is detailed in the "Request body" paragraph below.

##### Request command

The following HTTP request is sent from the provider to Ozwillo. 

<pre>
POST /apps/pending-instance/{instance_id} HTTP/1.1
Host: {kernel API host}
Accept: application/json, application/*+json
Authorization: Basic {base64 encoding of client_id:client_secret}
Content-Type: application/json;charset=UTF-8
</pre>

The host is typically either `kernel.ozwillo-preprod.eu` (preproduction) or `kernel.ozwillo.com` (production). The authorization header needs to be set as described in [Calling the Kernel without an access_token](#ref-2-3--2).

##### Request body

| Parameter name | Parameter description | Type |
| :-- | :-- | :-- |
| **instance_id** | Ozwillo application instance id | string |
| **services** | services (at least one) to be declared on Ozwillo, more details below | array of Service objects |
| **destruction_uri** | destruction endpoint of this instance | URI string |
| **destruction_secret** | secret used to compute the destruction request signature | URI string |
| needed_scopes | scopes needed by the instance | array of NeededScope objects |
| scopes | scopes declared by the instance | array of Scope objects |

**Embedded Service objects**

| Field name | Field description | Type |
| :-- | :-- | :-- |
| **local_id** | identifier that needs to be unique within the instance, for instance "front-end" | string |
| **name** | name in the default language (try to remain below 20 characters) | string |
| name#{lg} | name in the given language (try to remain below 20 characters) | string |
| **description** | description in the default language | markdown string |
| description#{lg} | description in the given language | markdown string |
| **tos_uri** | terms of service URI implicitly accepted on purchase, in the default language | URI string |
| tos_uri#{lg} | terms of service URI implicitly accepted on purchase, in the given language | URI string |
| **policy_uri** | privacy policy URI implicitly accepted on purchase, in the default language | URI string |
| policy_uri#{lg} | privacy policy URI implicitly accepted on purchase, in the given language | URI string |
| **icon** | service icon URI in the default language, a 64px x 64px png is expected | URI string |
| icon#{lg} | service icon URI in the given language, a 64px x 64px png is expected | URI string |
| screenshot_uris | list of screenshot URIs, a 850px x 450px png are expected | array of URI strings |
| **contacts** | list of URLs to contact the provider or its support | array of URI (URL or mailto) strings |
| **payment_option** | "FREE" or "PAID" setting | string |
| **target_audience** | is the service intended to "CITIZENS", "PUBLIC_BODIES" and/or "COMPANIES" | array of strings |
| category_ids | IDs of the service store categories, not supported for the moment | array of strings |
| visible | if false, the service is not visible in the application store; defaults to false | boolean |
| restricted | if true, the service restricts access to members (app_user and app_admin); defaults to false | boolean |
| **service_uri** | URL entrypoint of the service | URI string |
| notification_uri | endpoint used when there are notifications for this service | URI string |
| **redirect_uris** | whitelist of authentication callbacks, each URI must be unique to this service within the instance | array of URI strings |
| post_logout_redirect_uris | whitelist of post-logout callbacks, each URI must be unique to this service within the instance | array of URI strings |
| subscription_uri | endpoint of the subscription callback, usage is not implemented yet | array of URI strings |

**NB**: `redirect_uris` and `post_logout_redirect_uris` values can't be shared by different services within the same application instance. In some cases Ozwillo may identify a service thanks to either `instance_id` + `redirect_uri` or `instance_id` + `post_logout_redirect_uri` pairs. 
{: .focus .important}

A few remarks on this table:

- you may compare it with the declaration of [applications](#ref-3-1) in the catalog, and find many common features due to the fact both applications and services are [store entries](#def-store-entry);
- `redirect_uris` and `post_logout_redirect_uris` are related to [User authentication](#ref-4).

**Embedded NeededScope objects**

| Field name | Field description | Type |
| :-- | :-- | :-- |
| **scope_id** | full identifier of the needed scope | string |
| **motivation** | motivation for using the scope, in the default language | string |
| motivation#{lg} | motivation for using the scope, in the give language | string |

The motivation helps understand users why they should grant specific privileges (associated to the `scope_id) to the instance, and thus help them decide if they will.

**Embedded Scope objects**
{: #ref-3-2-3-scope}

| Field name | Field description | Type |
| :-- | :-- | :-- |
| **local_id** | local identifier of this scope (within the instance) | string |
| **name** | name of the scope in the default language | string |
| name#{lg} | name of the scope in the given language | string |
| **description** | description of the scope in the default language | string |
| description#{lg} | description of the scope in the given language | string |

Scope identifiers are simple strings that correspond to the permission scopes that client applications require to access part of this API's functionality. For instance, application X may define an API to create events within a particular instance Y; it would then declare the scope "addevent" of this instance.

When a third party instance Z wants to add an event to Y on behalf of a user, it needs to require the `{Y instance_id}:addevent` as part of its token negotiation (the user is asked to confirm that they want to give the "add events on Y" permission to Z).

The full identifier of an instance scope is in the form `{instance_id}:{local_id}`.

##### Response from Ozwillo

Possible status codes:

- 200: means the acknowledgement is valid and processed;
- <span id="ref-ack-422">422</span>: means there is a problem with the acknowledgement request, additional information about the particular error is given in the response body.

For successful 200 responses, the body response is in the form of:

<pre>
{
	"back-end" : "a15243e3-17a0-4511-b15c-c88e6784e287",
 	"front-end" : "31336385-f2ff-4488-8835-1f7da53669b9"
}
</pre>

Where `back-end` and `front-end` would be the `local_id` of two declared services within the instance, and the corresponding GUIDs being their internal Ozwillo ids.

##### Effects

At this stage:

- the application instance is created;
- its services are declared in the catalog;
- its scopes are declared and available for use by other applications;
- the purchaser has desktop shortcuts for the created services;
- `visible:true` services are listed in store.

#### #3bis Provider dismiss
{: #ref-3-2-4}

If the instance creation has failed for some reason during [step #2](#ref-3-2-2), the provider must issue a DELETE request on the instance registration URL so that Ozwillo does not show endless "pending" instance creation requests:

##### Request command

The following HTTP request is sent from the provider to Ozwillo. 

<pre>
DELETE /apps/pending-instance/{instance_id} HTTP/1.1
</pre>

#### Instance destruction
{: #ref-3-2-5}

Instance destruction is not part of the initial provisioning that occurs between steps #1 to #3, but will occur whenever an admin of the organization that purchased the application decides to destroy it.

##### Request command

The following HTTP request is sent from Ozwillo to the provider at the `destruction_uri` defined in [step #3](#ref-3-2-3) (decomposed in `destruction_path` and `destruction_host` below). 

<pre>
POST {destruction_path}
Host: {destruction_host}
Accept: application/json, application/*+json
X-Hub-Signature: sha1={HMAC-SHA1 digest of payload}
Content-Type: application/json;charset=UTF-8
</pre>

This scheme allows sharing the same destruction URI and secret among several instances, but Ozwillo also supports having separate destruction URI and secret for each instance. See more on how to check if this request reliably comes from Ozwillo depending on [verifying the signature](#ref-hmac-signature).

##### Request body

| Field name | Field description | Type |
| :-- | :-- | :-- |
| instance_id | identifier of the instance | string |

##### Response from provider

The destruction endpoint must respond with a successful status (200, 202 or 204) in a timely manner (not necessarily waiting for the underlying ressources to be actually released or archived). Ozwillo will then delete the instance (and its associated resources, like services) from its database and it will be impossible for users to authenticate to it.

If the request times out, the Kernel will delete the instance from its database nevertheless. Any (timely) non-successful status will abort the destruction (so it can be retried later).

### FAQ
{: #ref-3-3}

**What is the difference between cancellation and destruction URIs?**

Pending instances (those for which a request has been made to the app factory, but for which no [acknowledgement](#ref-3-2-3) has been sent) can be cancelled using the same mechanism as [Instance destruction](#ref-3-2-5) but at the application wide `cancellation_uri`.

So `cancellation_uri` (and associated secret) is linked to the app factory, when `destruction_uri` is linked to an application instance. It means in particular that `cancellation_uri` should reject the destruction of provisionned instances.

TODO: use cases examples.
{: .todo}