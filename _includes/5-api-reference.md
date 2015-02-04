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

- <span class="api-host">host</span> value is shortened to the host subdomain among `accounts`, `kernel` or `data` (read [more](#1-programming-interface))
- <span class="api-auth">auth</span> value is:
  - `basic` for basic auth**entication** with a valid `client_id`/`client_secret` pair (read [more](#2-auth-without-token))
  - `bearer` for bearer auth**orization** with a valid `access_token` (read [more](#2-auth-with-token))
  - dismissed if the endpoint requires no authorization
- <span class="api-input">input</span> describes a specific required input content type:
	- `JSON` for `Content-Type: application/json;charset=UTF-8`
	- `form` for `Content-Type: application/x-www-form-urlencoded`
- <span class="api-output">output</span> describes available output content types (`JWT` implying **signed** `JWT`)
- <span class="scopes">SCOPES</span> is filled when the endpoint response depends on these scopes being previously granted by the end-user and thus associated to an `access_token` (it means this **scopes** option will only appear for `bearer` **auth**)

#### Common behaviour
{: #s5-common-behaviour}

When trying to modify or delete a resource (`PUT` or `DELETE` command), it is expected that you've previously retrieved it (`GET` command) and got an associated `etag`.

Then you should create your `UPDATE` or `DELETE` request with an <a href="http://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.24" target="_blank">If-Match</a> header set with the etag.

It prevents from altering resources that have been modified elsewhere since the last time you accessed it. 

### Keys
{: #s5-api-keys}

The API reference is under work and can't be trusted as of today.
{: .warning}

Retrieve the public key used for OpenID Connect:

<div class="api-entry">
	<div class="api-command">GET /a/keys</div>
	<div class="api-options">
		<div class="api-request">
			<span class="api-host">accounts</span>
		</div>
		<div class="api-response">
			<span class="api-output">JWKS</span>
		</div>
	</div>
</div>

Returns a JSON Web Key Set containing the public key. See the <a href="https://tools.ietf.org/html/draft-ietf-jose-json-web-key-18" target="_blank">RFC</a> for more informations about JWKS.

<hr/>

### Users
{: #s5-api-users}

The API reference is under work and can't be trusted as of today.
{: .warning}

Return claims about the end-user:

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
		<code>openid profile email name address</code>
	</div>
</div>

<p>See the <a href="https://openid.net/specs/openid-connect-basic-1_0.html#UserInfo" target="_blank">OpenID Connect Draft</a>, the <a href="https://tools.ietf.org/html/draft-ietf-oauth-json-web-token-08" target="_blank">JWT Draft</a> and the <a href="https://tools.ietf.org/html/draft-ietf-jose-json-web-signature-11" target="_blank">JWS Draft</a> for more information.</p>

##### Response body

| Field name | Field description | Type |
| :-- | :-- | :-- |
| **sub** | ... | string |
| **name** | ... | string |
| **family_name** | ... | string |
| **given_name** | ... | string |
| **zoneinfo** | ... | string |
| **locale** | ... | string |
| **updated_at** | ... | string |
{: .request}

<hr/>

Return claims about the end-user:

<div class="api-entry">
	<div class="api-command">POST /a/userinfo</div>
	<div class="api-options">
		<div class="api-request">
			<span class="api-host">accounts</span>
			<span class="api-auth">bearer</span>
			<span class="api-input">JSON</span>
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

<p>See the <a href="https://openid.net/specs/openid-connect-basic-1_0.html#UserInfo" target="_blank">OpenID Connect Draft</a>, the <a href="https://tools.ietf.org/html/draft-ietf-oauth-json-web-token-08" target="_blank">JWT Draft</a> and the <a href="https://tools.ietf.org/html/draft-ietf-jose-json-web-signature-11" target="_blank">JWS Draft</a> for more information.</p>

##### Request body

| Field name | Field description | Type |
| :-- | :-- | :-- |
| **sub** | ... | string |
| **name** | ... | string |
| **family_name** | ... | string |
| **given_name** | ... | string |
| **zoneinfo** | ... | string |
| **locale** | ... | string |
| **updated_at** | ... | string |
{: .request}

<hr/>

### Authorization
{: #s5-api-authorization}

The API reference is under work and can't be trusted as of today.
{: .warning}

Grant authorizations to the client application:

<div class="api-entry">
	<div class="api-command">GET /a/auth</div>
	<div class="api-options">
		<div class="api-request">
			<span class="api-host">accounts</span>
		</div>
	</div>
</div>

See the <a href="https://tools.ietf.org/html/rfc6749#section-3.2" target="_blank">OAuth 2.0 RFC</a> and <a href="https://openid.net/specs/openid-connect-basic-1_0.html#ObtainingTokens" target="_blank">OpenID Connect RFC</a> for more information.

<hr/>

Grant authorizations to the client application:

<div class="api-entry">
	<div class="api-command">POST /a/auth</div>
	<div class="api-options">
		<div class="api-request">
			<span class="api-host">accounts</span>
			<span class="api-input">form</span>
		</div>
	</div>
</div>

See the <a href="https://tools.ietf.org/html/rfc6749#section-3.2" target="_blank">OAuth 2.0 RFC</a> and <a href="https://openid.net/specs/openid-connect-basic-1_0.html#ObtainingTokens" target="_blank">OpenID Connect RFC</a> for more information.

<hr/>

### Tokens
{: #s5-api-tokens}

The API reference is under work and can't be trusted as of today.
{: .warning}

Exchange an authorization code or a refresh token for an access token:

<div class="api-entry">
	<div class="api-command">POST /a/token</div>
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

See the <a href="https://tools.ietf.org/html/rfc6749#section-3.2" target="_blank">OAuth 2.0 RFC</a> and <a href="https://openid.net/specs/openid-connect-basic-1_0.html#ObtainingTokens" target="_blank">OpenID Connect RFC</a> for more information.

<hr/>

Get information about an access token:

<div class="api-entry">
	<div class="api-command">POST /a/tokeninfo</div>
	<div class="api-options">
		<div class="api-request">
			<span class="api-host">accounts</span>
			<span class="api-auth">bearer</span>
		</div>
		<div class="api-response">
			<span class="api-output">JSON</span>
		</div>
	</div>
</div>

<p>Token introspection endpoint: see the OAuth Token Introspection <a href="https://tools.ietf.org/html/draft-richer-oauth-introspection" target="_blank">draft</a> for more information.</p>

##### Response body

| Field name | Field description | Type |
| :-- | :-- | :-- |
| active | ... | boolean |
| exp | ... | integer |
| iat | ... | integer |
| scope | list of granted scopes | string (scopes separated by spaces) |
| client_id | ... | string |
| sub | ... | string |
| aud | ... | string |
| token_type | ... | string |
| sub_groups | ... | string |
{: .request}

<hr/>

Revoke a token:

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

See the <a href="https://tools.ietf.org/html/rfc7009" target="_blank">OAuth 2.0 Token Revocation RFC</a> for more information.

<hr/>

### Application instances
{: #s5-api-app-instances}

The API reference is under work and can't be trusted as of today.
{: .warning}

Acknowledge the provisioning of an instance:

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

#### Request body

<hr/>

Notify an error while provisioning the instance:

<div class="api-entry">
	<div class="api-command">DELETE /apps/pending-instance/{instance_id}</div>
	<div class="api-options">
		<div class="api-request">
			<span class="api-host">kernel</span>
			<span class="api-auth">basic</span>
		</div>
	</div>
</div>

#### Request body

<hr/>

Retrieve information on an application instance:

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

#### Response body

<hr/>

Destroy an application instance:

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

Retrieve the services of the application instance:

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

#### Response body

<hr/>

Add a new service to the application instance:

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

#### Request body

#### Response body

<hr/>

### Services
{: #s5-api-services}

The API reference is under work and can't be trusted as of today.
{: .warning}

Retrieve information about a service:

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

#### Response body

<hr/>

Update a service:

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

#### Request body

#### Response body

<hr/>

Delete a service:

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

#### Request body

#### Response body

<hr/>

### Acess control
{: #s5-api-access-control}

The API reference is under work and can't be trusted as of today.
{: .warning}

Ozwillo manages access control lists that links user to application instances.

Retrieve app_users of the app instance:

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

#### Reponse body

<hr/>

Add an app_user to the app instance:

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

#### Request body

#### Reponse body

<hr/>

Retrieve: an ACE:

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

### Response body

<hr/>

Delete an ACE:

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

### Response body

<hr/>

### Notifications
{: #s5-api-notifications}

The API reference is under work and can't be trusted as of today.
{: .warning}

Get all unread notifications for a defined user and a filter:

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

Publish a notification targeted to some users:

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

#### Request body

<hr/>

Change status (read, unread) of notifications for a user:

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

Subscribe to a typed event from the event bus:

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

Delete an event subscription:

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

Publish a typed event into the event bus:

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