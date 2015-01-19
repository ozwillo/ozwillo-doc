## Provisioning
{: #ref-3}

This section explains the provisioning protocol triggered between Ozwillo and the provider APIs when a [purchase act](#def-purchase-act) occurs on the portal. The outcome of a successful provisioning is an application instance that is both:

1. allocated and reachable on the provider servers;
2. declared with appropriate settings, especially regarding access rights, on Ozwillo;

where (2) enables the creation of [desk shortcuts](#def-desk-shortcuts) so that users can access (1).
{: .where}

The creation of an application instance on the provider side may imply configuration changes or new resources allocation. These implementation details are left unconstrained: the protocol focuses only on API communication. It means existing deployment operations does not have to be affected and should rather be exposed through this protocol.


### Prerequesite
{: #ref-3-2}

Since provisioning starts when a user performs a purchase act, it means the purchased application is declared in the [catalog](#def-catalog) **and** Ozwillo knows how to forward the request to the provider through its [App Factory](#def-app-factory).

Thus the following information is needed to have a well described *and* installable application in the catalog:

**NB**: From now on, displaying parameters in **boldface** means they are mandatory.
{: .focus .important}

#### Commercial information

| Field name | Field description | Field type and format |
| :-- | :-- | :-- |
| **name** | name in the default language (try to remain below 20 characters) | string |
| name#&lt;language&gt; | name in the given language (try to remain below 20 characters) | string |
| **description** | description in the default language | markdown string |
| description#&lt;language&gt; | description in the given language | markdown string |
| **tos_uri** | terms of service URI implicitly accepted on purchase, in the default language | URI string |
| tos_uri#&lt;language&gt; | terms of service URI implicitly accepted on purchase, in the given language | URI string |
| **policy_uri** | privacy policy URI implicitly accepted on purchase, in the default language | URI string |
| policy_uri#&lt;language&gt; | privacy policy URI implicitly accepted on purchase, in the given language | URI string |
| **icon** | app icon URI in the default language, a 64px x 64px png is expected | URI string |
| icon#&lt;language&gt; | app icon URI in the given language, a 64px x 64px png is expected | URI string |
| screenshot_uris | list of screenshot URIs, a 850px x 450px png are expected | array of URI strings |
| **contacts** | list of URLs to contact the provider or its support | array of URI (URL or mailto) strings |

A few comments on this table:

- the *default* language information is what will be displayed if the user preferred language information is not available. It can be whatever language suiting best your target audience;
- `<language>` is a two-letter code from the <a href="http://en.wikipedia.org/wiki/List_of_ISO_639-1_codes" target="_blank">ISO 639-1</a> nomenclature. Some examples regarding the fields introduced above: `name#es`, `description#it`.
- here is a description of the markdown <a href="http://daringfireball.net/projects/markdown/syntax" target="_blank">syntax</a>. In particular, you may include raw text separated by two line breaks to shape paragraphs.

#### Store filters

| Field name | Field description | Field type and format |
| :-- | :-- | :-- |
| **payment_option** | "FREE" or "PAID" setting | string |
| **target_audience** | is the application intended to "CITIZENS", "PUBLIC_BODIES" and/or "COMPANIES" | array of strings |
| visible | if false, the application is not visible in the application store | boolean |
| category_ids | IDs of the application store categories, not supported for the moment | array of strings |

#### App factory

| Field name | Field description | Field type and format |
| :-- | :-- | :-- |
| **instantiation_uri** | instantiation endpoint | URI string |
| **instantiation_secret** | secret used to compute the instantiation request signature | string |
| **cancellation_uri** | cancellation endpoint, to cancel pending instantiations | URI string |
| **cancellation_secret** | secret used to compute the cancellation request signature | string |

There is no official spec about secret strings, but 30 characters within a rich character set (richer than hexadecimal) can be seen as a bare minimum. Here is a <a href="https://www.grc.com/passwords.htm" target="_blank">tool</a> to generate random strings.

#### Summary

You will likely need to read the remainder of this section to fully understand how fields are used, especially those related to the App factory. It means these setup steps are required before receiving provisioning requests from Ozwillo:

1. gather all parameters info needed as described in the previous tables;
2. send it to <a mailto="providers@ozwillo.com">providers@ozwillo.com</a> along with your display name as an application provider (typically the name of your company);
3. you will be notified when the application is made available on the [preproduction](#ref-2-1).

To ease the process, you may submit a first and simplified version of the commercial info fields, focusing on simple versions of **required** fields, and resubmit it later with enriched contents.

### Protocol
{: #ref-3-3}

### Ozwillo request
{: #ref-3-3-1}

When a purchase act occurs, Ozwillo creates a new [application instance](def-application-instance) in its catalog, in a pending state (since it has not been provisioned for the moment). This instance is given a unique and constant `instance_id` and credentials (`client_id` and `client_secret`). The `client_secret` must remain to be known only by Ozwillo and the provider.

Then Ozwillo generates a request to the App factory `instantiation_uri` with the following parameters:

| Parameter name | Parameter description | Parameter type |
| :-- | :-- | :-- |
| **instance_id** | unique and constant identifier of the instance | string |
| **client_id** | used for authentication purposes | string |
| **client_secret** | used for authentication purposes | string |
| **user** | a description of the user that initiated the purchase act, containing at least its Ozwillo user id | JSON object |
| organization | a description of the organization if the purchase occured on behalf an organiation, containing at least Ozwillo organization id | JSON object |
| instance_registration_uri | the endpoint that must be used to acknowledge the provisioning back to Ozwillo | URI string |

The content of this request is encoded in JSON. A HMAC-SHA1 hash of the JSON string is computed using the application's `instantiated_secret` as key with an uppercase hexadecimal output format. This computed HMAC-SHA1 signature is then inserted in the X-Hub-Signature (as per <a href="https://pubsubhubbub.googlecode.com/git/pubsubhubbub-core-0.4.html#authednotify" target="_blank">PubSubHubbub Core 0.4</a>).

The provider must recompute the signature of the request body with the same secret key and method. If signatures match, this reasonably certifies the instantiation request comes from Ozwillo (which is the only one knowing the `instantiated_secret`) and then can be trusted. If not, you may store the request for further analysis, and ignore it.

### Provider provisioning
{: #ref-3-3-2}

Provider provisioning: implementation agnostic, store request.