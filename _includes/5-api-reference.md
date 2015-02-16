## API Reference
{: #s5-api-reference}

### Introduction
{: #s5-introduction}

The API reference is under work and can't be trusted as of today.
{: .warning}

#### Endpoints description
{: #s5-endpoints-description}

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
{: #s5-common-behaviour}

When trying to modify or delete a resource (`PUT` or `DELETE` command), it is expected that you've previously retrieved it (`GET` command) and got an associated ["ETag"](https://tools.ietf.org/html/rfc7232#section-2.3) (entity-tags are also returned in some responses).

Then you should create your `UPDATE` or `DELETE` request with an <a href="https://tools.ietf.org/html/rfc7232#section-3.1" target="_blank">If-Match</a> header set with the entity-tag.

It prevents from altering resources that have been modified elsewhere since the last time you accessed it. 

### Keys
{: #s5-api-keys}

The API reference is under work and can't be trusted as of today.
{: .warning}

#### Retrieve the public key used for OpenID Connect

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

Returns a JSON Web Key Set containing the public key. See the <a href="https://tools.ietf.org/html/draft-ietf-jose-json-web-key-18" target="_blank">RFC</a> for more informations about JWKS.

Note that this API requires Basic authentication not for security concerns, but actually only so we can track who calls it and at which frequency.

<hr/>

### Users
{: #s5-api-users}

The API reference is under work and can't be trusted as of today.
{: .warning}

#### Return claims about the end-user

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

<p>See the <a href="https://openid.net/specs/openid-connect-basic-1_0.html#UserInfo" target="_blank">OpenID Connect Draft</a>, the <a href="https://tools.ietf.org/html/draft-ietf-oauth-json-web-token-08" target="_blank">JWT Draft</a> and the <a href="https://tools.ietf.org/html/draft-ietf-jose-json-web-signature-11" target="_blank">JWS Draft</a> for more information.</p>

##### Response body

| Field name | Field description | Type | Included if scope |
| :-- | :-- | :-- | :-- |
| **sub** | Identifier of the user | string | `openid` |
| name | User's full name in displayable form | string | `profile` |
| given_name | Given name(s) or first name(s) of the user | string | `profile` |
| family_name | Surname(s) or last name(s) of the user | string | `profile` |
| middle_name | Middle name(s) of the user | string | `profile` |
| nickname | Casual name of the user | string | `profile` |
| picture | URL of the user's profile picture (avatar) | URI | `profile` |
| gender | User's gender; either `male` or `female` | string | `profile` |
| birthdate | User's birthday, in ISO 8601 `YYYY-MM-DD` format | string | `profile` |
| zoneinfo | User's timezone, as a value from the [`tz` database](http://www.twinsun.com/tz/tz-link.htm) | string | `profile` |
| locale | User's locale, as a [BCP47](https://tools.ietf.org/html/bcp47) language tag | string | `profile` |
| email | User's e-mail address | string | `email` |
| email_verified | True if the user's e-mail address has been verified; otherwise false. | boolean | `email` |
| address | User's postal address | Address object | `address` |
| phone_number | User's telephone number | string | `phone` |
| phone_number_verfied | True if the user's phone number has been verified; potherwise false. | boolean | `phone` |
| **updated_at** | Time the user's information was last updated, in seconds since Unix Epoch | number | `openid` |
{: .request}

<hr/>

#### Return claims about the end-user

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

See the <a href="https://openid.net/specs/openid-connect-basic-1_0.html#UserInfo" target="_blank">OpenID Connect Draft</a>, the <a href="https://tools.ietf.org/html/draft-ietf-oauth-json-web-token-08" target="_blank">JWT Draft</a> and the <a href="https://tools.ietf.org/html/draft-ietf-jose-json-web-signature-11" target="_blank">JWS Draft</a> for more information.

<hr/>

### Authorization
{: #s5-api-authorization}

The API reference is under work and can't be trusted as of today.
{: .warning}

#### Grant authorizations to the client application

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
{: #s5-api-tokens}

The API reference is under work and can't be trusted as of today.
{: .warning}

#### Exchange an authorization code or a refresh token for an access token

<div class="api-entry">
	<div class="api-command">POST /a/token</div>
	<div class="api-options">
		<div class="api-request">
			<span class="api-host">accounts</span>
			<span class="api-auth">bearer</span>
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

<div class="api-entry">
	<div class="api-command">POST /a/tokeninfo</div>
	<div class="api-options">
		<div class="api-request">
			<span class="api-host">accounts</span>
			<span class="api-auth">bearer</span>
			<span class="api-input">form</span>
		</div>
		<div class="api-response">
			<span class="api-output">JSON</span>
		</div>
	</div>
</div>

See <a href="https://tools.ietf.org/html/draft-richer-oauth-introspection" target="_blank">OAuth 2.0 Token Introspection</a> for more information.

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

<div class="api-entry">
	<div class="api-command">POST /a/revoke</div>
	<div class="api-options">
		<div class="api-request">
			<span class="api-host">accounts</span>
			<span class="api-auth">bearer</span>
			<span class="api-input">form</span>
		</div>
		<div class="api-response">
			<span class="api-output">JWT</span>
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
{: #s5-api-app-instances}

The API reference is under work and can't be trusted as of today.
{: .warning}

#### Acknowledge the provisioning of an instance

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

##### Request body

<hr/>

#### Notify an error while provisioning the instance

<div class="api-entry">
	<div class="api-command">DELETE /apps/pending-instance/{instance_id}</div>
	<div class="api-options">
		<div class="api-request">
			<span class="api-host">kernel</span>
			<span class="api-auth">basic</span>
		</div>
	</div>
</div>

##### Request body

<hr/>

#### Retrieve information on an application instance

<div class="api-entry">
	<div class="api-command">GET /apps/instance/{instance_id}</div>
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

<hr/>

#### Destroy an application instance

<div class="api-entry">
	<div class="api-command">DELETE /apps/instance/{instance_id}</div>
	<div class="api-options">
		<div class="api-request">
			<span class="api-host">kernel</span>
			<span class="api-auth">bearer</span>
		</div>
	</div>
</div>

If-Match

<hr/>

#### Retrieve the services of the application instance

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

<hr/>

#### Add a new service to the application instance

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

##### Response body

<hr/>

### Services
{: #s5-api-services}

The API reference is under work and can't be trusted as of today.
{: .warning}

#### Retrieve information about a service

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

<hr/>

#### Update a service

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

If-Match

##### Request body

##### Response body

<hr/>

#### Delete a service

<div class="api-entry">
	<div class="api-command">DELETE /apps/service/{service_id}</div>
	<div class="api-options">
		<div class="api-request">
			<span class="api-host">kernel</span>
			<span class="api-auth">bearer</span>
		</div>
	</div>
</div>

If-Match

##### Request body

##### Response body

<hr/>

### Access control
{: #s5-api-access-control}

The API reference is under work and can't be trusted as of today.
{: .warning}

Ozwillo manages access control lists that links user to application instances.

#### Retrieve app_users of the app instance

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

TODO: only for app_admins?
{: .todo}

##### Reponse body

<hr/>

#### Add an app_user to the app instance

<div class="api-entry">
	<div class="api-command">POST /apps/acl/instance/{instance_id}</div>
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

TODO: only for app_admins?
{: .todo}

##### Request body

##### Reponse body

<hr/>

#### Retrieve an ACE

<div class="api-entry">
	<div class="api-command">GET /apps/acl/ace/{ace_id}</div>
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

<hr/>

#### Delete an ACE

<div class="api-entry">
	<div class="api-command">DELETE /apps/acl/ace/{ace_id}</div>
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

<hr/>

### Notifications
{: #s5-api-notifications}

The API reference is under work and can't be trusted as of today.
{: .warning}

#### Get all unread notifications for a defined user and a filter

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

It will give you the notifications for the given user and app-instance; note that the instance_id must be the one for which you obtained the access_token. You can also pass an additional `status=READ` or `status=UNREAD` query-string parameter to filter notifications to only those that have been read or are still unread.

##### Response body

| Field name | Field description | Type |
| :-- | :-- | :-- |
| Notification | ... | object |
| Instant | ... | object |
| Chronology | ... | object |
| DateTimeZone | list of granted scopes | object |
{: .request}

<p>JSON payload like { message_ids: [<array of message IDs], status: "READ" } allows you to change the status of the given notifications (status can obviously also be "UNREAD" if needed).</p>

TODO: detail Notification, Instant, Chronology, DateTimeZone
{: .todo}

<hr/>

#### Publish a notification targeted to some users

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

<hr/>

#### Change status (read, unread) of notifications for a user

<div class="api-entry">
	<div class="api-command">POST /n/{user_id}/messages</div>
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

<hr/>

### Events
{: #s5-api-events}

The API reference is under work and can't be trusted as of today.
{: .warning}

#### Subscribe to a typed event from the event bus

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