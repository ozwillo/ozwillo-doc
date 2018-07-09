## API Reference
{: #s6-api-reference}

#### Endpoints description
{: #s6-endpoints-description}

As a reminder the APIs are served accross several hosts depending on their topic (authentication, provisioning, data...). Endpoints descriptions follow this format:

<div class="api-entry">
	<div class="api-command">{method} {path}</div>
	<div class="api-options">
		<div class="api-request">
			<span class="api-host">host</span>
			<span class="api-auth">auth</span>
			<span class="api-input">input</span>
		</div>
		<div class="api-response">
			<span class="api-output">output</span>
		</div>
	</div>
	<div class="api-scopes">
		<span class="scopes">SCOPES</span>
		<code>{scopes}</code>
	</div>
</div>

Where:

- <span class="api-host">host</span> value is shortened to the host subdomain among `accounts`, `kernel` or `data` (read [more](#s1-programming-interface))
- <span class="api-auth">auth</span> value is:
  - `basic` for basic auth**entication** with a valid `client_id`/`client_secret` pair (read [more](#s2-auth-without-token))
  - `bearer` for bearer auth**orization** with a valid `access_token` (read [more](#s2-auth-with-token))
  - dismissed if the endpoint requires no authorization
- <span class="api-input">input</span> describes a specific required input content type:
	- `JSON` for `Content-Type: application/json;charset=UTF-8`
	- `form` for `Content-Type: application/x-www-form-urlencoded`
- <span class="api-output">output</span> describes available output content types (`JWT` implying **signed** `JWT`)
- <span class="scopes">SCOPES</span> is filled when the endpoint response depends on these scopes being previously granted by the end-user and thus associated to an `access_token` (it means this **scopes** option will only appear for `bearer` **auth**)

#### Common behaviour
{: #s6-common-behaviour}

When trying to modify or delete a resource (`PUT` or `DELETE` command), it is expected that you've previously retrieved it (`GET` command) and got an associated ["ETag"](https://tools.ietf.org/html/rfc7232#section-2.3) (entity-tags are also returned in some responses).

Then you should create your `UPDATE` or `DELETE` request with an <a href="https://tools.ietf.org/html/rfc7232#section-3.1" target="_blank">If-Match</a> header set with the entity-tag.

It prevents from altering resources that have been modified elsewhere since the last time you accessed it. 

#### Datacore-specific behaviour
{: #s6-datacore-behaviour}

In addition to common behaviour:

- when trying to get a resource (`GET` command), in order to benefit from your client-side cache if any, provide an <a href="https://tools.ietf.org/html/rfc7232#section-3.2">If-None-Match</a> header set with the version you have in said cache as entity-tag.
- when attempting any change (`POST`, `PUT` or `DELETE` command), it is mandatory to provide the project they (should) reside in using the X-Datacore-Project HTTP header. Otherwise, the current project will be the default `oasis.main` one. Therefore it is also mandatory if resources reside in a project that is not among oasis.main's visible projects, such as `oasis.sandbox` or forks. So it is an overall good practice.
- providing the `X-Datacore-View` header allows to get back fewer data, in order to optimize bandwidth use.

And again, the best reference for the Datacore API is its live [Swagger technical Playground (preproduction)](https://data.ozwillo-preprod.eu/dc-ui/index.html#swagger).

### Keys
{: #s6-api-keys}

#### Retrieve the public key used for OpenID Connect
{: #s6-get-a-keys}

<div class="api-entry">
	<div class="api-command">GET /a/keys</div>
	<div class="api-options">
		<div class="api-request">
			<span class="api-host">accounts</span>
			<span class="api-auth">basic</span>
		</div>
		<div class="api-response">
			<span class="api-output">JWKS</span>
		</div>
	</div>
</div>

Returns a JSON Web Key Set containing the public key. See <a href="https://tools.ietf.org/html/rfc7517" target="_blank">JWK (RFC 7517)</a> for more informations about JWKS.

Note that this API requires Basic authentication not for security concerns, but actually only so we can track who calls it and at which frequency. _Note: this is no longer the case starting with v1.19._

<hr/>

### Users
{: #s6-api-users}

#### Return claims about the end-user
{: #s6-get-a-userinfo}

<div class="api-entry">
	<div class="api-command">GET /a/userinfo</div>
	<div class="api-options">
		<div class="api-request">
			<span class="api-host">accounts</span>
			<span class="api-auth">bearer</span>
		</div>
		<div class="api-response">
			<span class="api-output">JSON</span>
			<span class="api-output">JWT</span>
		</div>
	</div>
	<div class="api-scopes">
		<span class="scopes">SCOPES</span>
		<code>openid</code> and optionally any of <code>profile email name address</code>
	</div>
</div>

<p>See <a href="https://openid.net/specs/openid-connect-basic-1_0.html#UserInfo" target="_blank">OpenID Connect 1.0</a>, <a href="https://tools.ietf.org/html/rfc7519" target="_blank">JWT (RFC 7519)</a> and <a href="https://tools.ietf.org/html/rfc7515" target="_blank">JWS (RFC 7515)</a> for more information.</p>

##### Response body

| Field name | Field description | Type | Included if scope |
| :-- | :-- | :-- | :-- |
| **sub** | Identifier of the user | string | `openid` |
| name | User's full name in displayable form | string | `profile` |
| given_name | Given name(s) or first name(s) of the user | string | `profile` |
| family_name | Surname(s) or last name(s) of the user | string | `profile` |
| middle_name | Middle name(s) of the user | string | `profile` |
| nickname | Casual name of the user | string | `profile` |
| gender | User's gender; either `male` or `female` | string | `profile` |
| birthdate | User's birthday, in ISO 8601 `YYYY-MM-DD` format | string | `profile` |
| locale | User's locale, as a [BCP47](https://tools.ietf.org/html/bcp47) language tag | string | `profile` |
| email | User's e-mail address | string | `email` |
| email_verified | True if the user's e-mail address has been verified; otherwise false. | boolean | `email` |
| address | User's postal address | Address object | `address` |
| phone_number | User's telephone number | string | `phone` |
| phone_number_verified | True if the user's phone number has been verified; potherwise false. | boolean | `phone` |
| **updated_at** | Time the user's information was last updated, in seconds since Unix Epoch | number | `openid` |
{: .request}

<hr/>

#### Return claims about the end-user
{: #s6-post-a-userinfo}

<div class="api-entry">
	<div class="api-command">POST /a/userinfo</div>
	<div class="api-options">
		<div class="api-request">
			<span class="api-host">accounts</span>
			<span class="api-auth">bearer</span>
		</div>
		<div class="api-response">
			<span class="api-output">JSON</span>
			<span class="api-output">JWT</span>
		</div>
	</div>
	<div class="api-scopes">
		<span>SCOPES</span>
		<code>openid profile email name address</code>
	</div>
</div>

Same as above, only the HTTP method changes.

See <a href="https://openid.net/specs/openid-connect-basic-1_0.html#UserInfo" target="_blank">OpenID Connect 1.0</a>, <a href="https://tools.ietf.org/html/rfc7519" target="_blank">JWT (RFC 7519)</a> and <a href="https://tools.ietf.org/html/rfc7515" target="_blank">JWS (RFC 7515)</a> for more information.

<hr/>

### Authorization
{: #s6-api-authorization}

#### Grant authorizations to the client application
{: #s6-get-a-auth}

<div class="api-entry">
	<div class="api-command">GET /a/auth</div>
	<div class="api-options">
		<div class="api-request">
			<span class="api-host">accounts</span>
		</div>
	</div>
</div>

See <a href="https://tools.ietf.org/html/rfc6749#section-3.2" target="_blank">OAuth 2.0</a> and <a href="https://openid.net/specs/openid-connect-basic-1_0.html#AuthenticationRequest" target="_blank">OpenID Connect 1.0</a> for more information.

<hr/>

#### Grant authorizations to the client application
{: #s6-post-a-auth}

<div class="api-entry">
	<div class="api-command">POST /a/auth</div>
	<div class="api-options">
		<div class="api-request">
			<span class="api-host">accounts</span>
			<span class="api-input">form</span>
		</div>
	</div>
</div>

Same as above, only the HTTP method changes, and parameters are sent in the request payload rather than the query string.

See <a href="https://tools.ietf.org/html/rfc6749#section-3.2" target="_blank">OAuth 2.0</a> and <a href="https://openid.net/specs/openid-connect-basic-1_0.html#AuthenticationRequest" target="_blank">OpenID Connect 1.0</a> for more information.

<hr/>

### Tokens
{: #s6-api-tokens}

#### Exchange an authorization code or a refresh token for an access token
{: #s6-post-a-token}

<div class="api-entry">
	<div class="api-command">POST /a/token</div>
	<div class="api-options">
		<div class="api-request">
			<span class="api-host">accounts</span>
			<span class="api-auth">basic</span>
			<span class="api-input">form</span>
		</div>
		<div class="api-response">
			<span class="api-output">JSON</span>
		</div>
	</div>
</div>

See <a href="https://tools.ietf.org/html/rfc6749#section-3.2" target="_blank">OAuth 2.0</a> and <a href="https://openid.net/specs/openid-connect-basic-1_0.html#ObtainingTokens" target="_blank">OpenID Connect 1.0</a> for more information.

<hr/>

#### Get information about an access token
{: #s6-post-a-tokeninfo}

<div class="api-entry">
	<div class="api-command">POST /a/tokeninfo</div>
	<div class="api-options">
		<div class="api-request">
			<span class="api-host">accounts</span>
			<span class="api-auth">basic</span>
			<span class="api-input">form</span>
		</div>
		<div class="api-response">
			<span class="api-output">JSON</span>
		</div>
	</div>
</div>

See <a href="https://tools.ietf.org/html/rfc7662" target="_blank">OAuth 2.0 Token Introspection (RFC 7662)</a> for more information.

This endpoint is only accessible to _protected resources_ (in OAuth 2.0 parlance) that need to validate whether a received `access_token` is valid and retrieve information about it. Other clients will get an `{ "active": false }` response; even the app-instance for which the token has been issued (this might change in the future though).

##### Request body

| Field name | Field description | Type |
| :-- | :-- | :-- |
| **token** | The string value of the token to introspect. | string |
{: .request}

##### Response body

| Field name | Field description | Type |
| :-- | :-- | :-- |
| **active** | Indicator of whether or not the presented token is currently active | boolean |
| scope | Space-separated list of scopes associated with this token | string |
| client_id | Identifier of the app-instance for which the token was issued | string |
| token_type | Type of the token (e.g. `Bearer`) | string |
| exp | Timestamp (in seconds since Unix Epoch) indicating when the token will expire | integer |
| iat | Timestamp (in seconds since Unix Epoch) indicating when the token was originally issued | integer |
| nbf | Timestamp (in seconds since Unix Epoch) indicating when this token is not to be used before | integer |
| sub | Identifier of the user who authorized this token. | string |
{: .request}

{::comment}Document sub_groups once we have proper (and stable) support for user groups.{:/}

<hr/>

#### Revoke a token
{: #s6-post-a-revoke}

<div class="api-entry">
	<div class="api-command">POST /a/revoke</div>
	<div class="api-options">
		<div class="api-request">
			<span class="api-host">accounts</span>
			<span class="api-auth">basic</span>
			<span class="api-input">form</span>
		</div>
	</div>
</div>

See <a href="https://tools.ietf.org/html/rfc7009" target="_blank">OAuth 2.0 Token Revocation (RFC 7009)</a> for more information.

> Note: invalid tokens do not cause an error response since the client cannot handle such an error in a reasonable way. Moreover, the purpose of the revocation request, invalidating the particular token, is already achieved.

##### Request body

| Field name | Field description | Type |
| :-- | :-- | :-- |
| **token** | The string value of the token to introspect. | string |
{: .request}

<hr/>

### Application instances
{: #s6-api-app-instances}

#### Acknowledge the provisioning of an instance
{: #s6-post-apps-pending-instance}

<div class="api-entry">
	<div class="api-command">POST /apps/pending-instance/{instance_id}</div>
	<div class="api-options">
		<div class="api-request">
			<span class="api-host">kernel</span>
			<span class="api-auth">basic</span>
			<span class="api-input">JSON</span>
		</div>
	</div>
</div>

See [above](#s3-3-provider-acknowledgement) for details.

<hr/>

#### Notify an error while provisioning the instance
{: #s6-delete-apps-pending-instance}

<div class="api-entry">
	<div class="api-command">DELETE /apps/pending-instance/{instance_id}</div>
	<div class="api-options">
		<div class="api-request">
			<span class="api-host">kernel</span>
			<span class="api-auth">basic</span>
		</div>
	</div>
</div>

See [above](s3-3bis-provider-dismiss) for details.

<hr/>

#### Retrieve the services of the application instance
{: #s6-get-apps-instance-id-services}

<div class="api-entry">
	<div class="api-command">GET /apps/instance/{instance_id}/services</div>
	<div class="api-options">
		<div class="api-request">
			<span class="api-host">kernel</span>
			<span class="api-auth">bearer</span>
		</div>
		<div class="api-response">
			<span class="api-output">JSON</span>
		</div>
	</div>
</div>

##### Response body

The response is a JSON Array of [service objects](#s3-3-provider-acknowledgement-service) as used during provisioning.

<hr/>

#### Add a new service to the application instance
{: #s6-post-apps-instance-id-services}

<div class="api-entry">
	<div class="api-command">POST /apps/instance/{instance_id}/services</div>
	<div class="api-options">
		<div class="api-request">
			<span class="api-host">kernel</span>
			<span class="api-auth">bearer</span>
			<span class="api-input">JSON</span>
		</div>
		<div class="api-response">
			<span class="api-output">JSON</span>
		</div>
	</div>
</div>

##### Request body

See [embedded service objects](#s3-3-provider-acknowledgement-service) used during provisioning.

<hr/>

### Services
{: #s6-api-services}

#### Retrieve information about a service
{: #s6-get-apps-service}

<div class="api-entry">
	<div class="api-command">GET /apps/service/{service_id}</div>
	<div class="api-options">
		<div class="api-request">
			<span class="api-host">kernel</span>
			<span class="api-auth">bearer</span>
		</div>
		<div class="api-response">
			<span class="api-output">JSON</span>
		</div>
	</div>
</div>

##### Response body

See [embedded service objects](#s3-3-provider-acknowledgement-service) used during provisioning.

<hr/>

#### Update a service
{: #s6-put-apps-service}

<div class="api-entry">
	<div class="api-command">PUT /apps/service/{service_id}</div>
	<div class="api-options">
		<div class="api-request">
			<span class="api-host">kernel</span>
			<span class="api-auth">bearer</span>
			<span class="api-input">JSON</span>
		</div>
		<div class="api-response">
			<span class="api-output">JSON</span>
		</div>
	</div>
</div>

##### Request body

See [embedded service objects](#s3-3-provider-acknowledgement-service) used during provisioning.

Note that the provided description _replaces_ the one stored in Ozwillo. This is not a _patch_ or _partial update._

<hr/>

#### Delete a service
{: #s6-delete-apps-service}

<div class="api-entry">
	<div class="api-command">DELETE /apps/service/{service_id}</div>
	<div class="api-options">
		<div class="api-request">
			<span class="api-host">kernel</span>
			<span class="api-auth">bearer</span>
		</div>
	</div>
</div>

<hr/>

### Access control
{: #s6-api-access-control}

Ozwillo manages access control lists that links user to application instances.

#### Retrieve app_users (and app_admins) of the app instance
{: #s6-get-apps-acl-instance}

<div class="api-entry">
	<div class="api-command">GET /apps/acl/instance/{instance_id}</div>
	<div class="api-options">
		<div class="api-request">
			<span class="api-host">kernel</span>
			<span class="api-auth">bearer</span>
		</div>
		<div class="api-response">
			<span class="api-output">JSON</span>
		</div>
	</div>
</div>

This API is only available to users who are already `app_admin`s for the application instance.

##### Response body

The response is a JSON Array of access control entries, each with the following fields:

| Field name | Field description | Type |
| :-- | :-- | :-- |
| **instance_id** | Identifier for the application instance | string |
| **user_id** | Identifier for the user whom the entry is about | string |
| user_name | (non-unique) display name for the user whom the entry is about | string |
| creator_id | Identifier for the user who created the entry | string |
| creator_name | (non-unique) display name for the user who created the entry | string |
| app_user | Whether the `user_id` is an `app_user` for the instance | boolean |
| app_admin | Whether the `user_id` is an `app_admin` for the instance | boolean |

At least one of `app_user` or `app_admin` will be present and `true`. Both might be `true` for a given entry.

<hr/>

### Notifications
{: #s6-api-notifications}

#### Get all unread notifications for a defined user and a filter
{: #s6-get-n-id-messages}

<div class="api-entry">
	<div class="api-command">GET /n/{user_id}/messages?instance={instance_id}</div>
	<div class="api-options">
		<div class="api-request">
			<span class="api-host">kernel</span>
			<span class="api-auth">bearer</span>
		</div>
		<div class="api-response">
			<span class="api-output">JSON</span>
		</div>
	</div>
</div>

It will give you the notifications for the given user and app-instance; note that the `instance_id` must be the one for which you obtained the `access_token`. You can also pass an additional `status=READ` or `status=UNREAD` query-string parameter to filter notifications to only those that have been read or are still unread.

##### Response body

The response is a JSON Array of messages, each with the following fields:

| Field name | Field description | Type |
| :-- | :-- | :-- |
| **id** | Identifier for the message | string |
| **user_id** | Identifier of the user that received the message | string |
| **instance_id** | Identifier of the app-instance that emitted the message | string |
| service_id | Identifier of the service that emitted the message | string |
| **message** | Text of the message | string |
| message#{l} | Localized text of the message | string |
| action_uri | URI for the call to action | URI |
| action_uri#{l} | Localized URI for the call to action | URI |
| action_label | Label of the call to action | string |
| action_label#{l} | Localized label of the call to action | string |
| **time** | Timestamp at which the message was emitted, in milliseconds since Unix Epoch | number |
| **status** | Read status of the message; either `READ` or `UNREAD` | string |
{: .request}

<hr/>

#### Change read status of a user's notification messages
{: #s6-post-n-id-messages}

<div class="api-entry">
	<div class="api-command">POST /n/{user_id}/messages</div>
	<div class="api-options">
		<div class="api-request">
			<span class="api-host">kernel</span>
			<span class="api-auth">bearer</span>
			<span class="api-input">JSON</span>
		</div>
	</div>
</div>

##### Request body

| Field name | Field description | Type |
| :-- | :-- | :-- |
| **message_ids** | Identifiers of the message to update | array of strings |
| **status** | New status for the identified messages; either `READ` or `UNREAD` | string |
{: .request}

<hr/>

#### Publish a notification targeted to some users
{: #s6-post-n-publish}

<div class="api-entry">
	<div class="api-command">POST /n/publish</div>
	<div class="api-options">
		<div class="api-request">
			<span class="api-host">kernel</span>
			<span class="api-auth">bearer</span>
			<span class="api-input">JSON</span>
		</div>
		<div class="api-response">
			<span class="api-output">JSON</span>
		</div>
	</div>
</div>

##### Request body

| Field name | Field description | Type |
| :-- | :-- | :-- |
| **user_ids** | Identifiers of the users to notify | array of strings |
| **service_id** | Identifier of the service that emits the message | string |
| **message** | Text of the message | string |
| message#{l} | Localized text of the message | string |
| action_uri | URI for the call to action | URI |
| action_uri#{l} | Localized URI for the call to action | URI |
| action_label | Label of the call to action | string |
| action_label#{l} | Localized label of the call to action | string |
{: .request}

<hr/>

### Events
{: #s6-api-events}

The Events API documentation is under work and can't be trusted as of today.
{: .warning}

#### Subscribe to a typed event from the event bus
{: #s6-post-e-id-subscriptions}

<div class="api-entry">
	<div class="api-command">POST /e/{instance_id}/subscriptions</div>
	<div class="api-options">
		<div class="api-request">
			<span class="api-host">kernel</span>
			<span class="api-auth">bearer</span>
			<span class="api-input">JSON</span>
		</div>
	</div>
</div>

The returned location URL get access to the subscription (delete the subscription)

##### Request body

<hr/>

#### Delete an event subscription
{: #s6-delete-e-subscription}

<div class="api-entry">
	<div class="api-command">DELETE /e/subscription/{subscription_id}</div>
	<div class="api-options">
		<div class="api-request">
			<span class="api-host">kernel</span>
			<span class="api-auth">bearer</span>
		</div>
	</div>
</div>

If-Match

<hr/>

#### Publish a typed event into the event bus
{: #s6-post-e-publish}

<div class="api-entry">
	<div class="api-command">POST /e/publish</div>
	<div class="api-options">
		<div class="api-request">
			<span class="api-host">kernel</span>
			<span class="api-auth">bearer</span>
			<span class="api-input">JSON</span>
		</div>
	</div>
</div>

##### Request body

<hr/>

### Data Resources
{: #s6-api-data-resources}

This endpoint allows to manage Datacore Resources (CRUD : Create, Read, Update, Delete) in typical RESTful fashion (GET, POST, PUT, DELETE, not respectively). Using it requires the `datacore` [scope](#s1-scopes).

Have a look at the Datacore's [Resources API reference](https://data.ozwillo-preprod.eu/dc-ui/index.html#!/dc) in its live Swagger technical Playground ([how to get access](#s2-online-experimentation)) and try it out there.

The most common operations are, in typical RESTful fashion:

- [GET /dc/type/{type}](https://data.ozwillo-preprod.eu/dc/findDataInType_get_2) for typed queries (criteria including $fulltext, sort, ranged query- or iterator-based pagination, debug mode, view),
- [POST /dc/type/{type}](https://data.ozwillo-preprod.eu/dc/postAllDataInType_post_0) for typed creation.

Also useful are:

- its companion [PUT /dc/type/{type}](https://data.ozwillo-preprod.eu/dc/putAllDataInType_put_1) for updates (mind the version),
- [GET /dc/type/{type}](https://data.ozwillo-preprod.eu/dc/getData_get_7) (mind the version to benefit from your client-side cache if any),
- and [DELETE /dc/type/{type}](https://data.ozwillo-preprod.eu/dc/deleteData_delete_8) (mind the version).
<br/><br/>

Note that Datacore Models are themselves stored as Resources having the dcmo:model_0 (meta) model in the oasis.meta project.

<hr/>

### Data Rights
{: #s6-api-data-rights}

This endpoint allows to manage rights of Datacore Resources at the Resource-level, i.e. for each one of them their owners (required to be able to use the Rights API), writers (also allows to delete), and readers beyond default owners, writers and readers specified at Model or by default Project level. Using it requires the `datacore` [scope](#s1-scopes).

Have a look at the Datacore's [Rights API reference](https://data.ozwillo-preprod.eu/dc-ui/index.html#!/r) in its live Swagger technical Playground ([how to get access](#s2-online-experimentation)) and try it out there.

<hr/>

### Data History
{: #s6-api-data-history}

This endpoint allows to retrieve older versions of Resources, since the time when historization was enabled ("dcmo:isHistorizable" : false in its model). Using it requires the `datacore` [scope](#s1-scopes).

Have a look at the Datacore's [History API reference](https://data.ozwillo-preprod.eu/dc-ui/index.html#!/dc/findHistorizedResource_get_13) in its live Swagger technical Playground ([how to get access](#s2-online-experimentation)) and try it out there.

<hr/>

### Data Contributions
{: #s6-api-data-contributions}

This endpoint allows users that don't have write permissions to contribute new versions of existing resources to be approved by their owner. Using it requires the `datacore` [scope](#s1-scopes).

Have a look at the Datacore's [Contributions API reference](https://data.ozwillo-preprod.eu/dc-ui/index.html#!/c) in its live Swagger technical Playground ([how to get access](#s2-online-experimentation)) and try it out there.