## API Reference
{: #s5-api-reference}

The API reference is under work and can't be trusted as of today.
{: .warning}

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
  - missing if the endpoint requires no authorization
- <span class="api-input">input</span> describes a specific required input content type:
	- `JSON` for `Content-Type: application/json;charset=UTF-8`
	- `form` for `Content-Type: application/x-www-form-urlencoded`
- <span class="api-output">output</span> describes available output content types (`JWT` implying **signed** `JWT`)
- <span class="scopes">SCOPES</span> is filled when the endpoint response depends on these scopes being previously granted by the end-user and thus associated to an `access_token` (it means this **scopes** option will only appear for `bearer` **auth**)

### Users
{: #s5-api-users}

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

<p>See the <a href="http://openid.net/specs/openid-connect-basic-1_0.html#UserInfo" target="_blank">OpenID Connect Draft</a>, the <a href="http://tools.ietf.org/html/draft-ietf-oauth-json-web-token-08" target="_blank">JWT Draft</a> and the <a href="http://tools.ietf.org/html/draft-ietf-jose-json-web-signature-11" target="_blank">JWS Draft</a> for more information.</p>

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

<p>See the <a href="http://openid.net/specs/openid-connect-basic-1_0.html#UserInfo" target="_blank">OpenID Connect Draft</a>, the <a href="http://tools.ietf.org/html/draft-ietf-oauth-json-web-token-08" target="_blank">JWT Draft</a> and the <a href="http://tools.ietf.org/html/draft-ietf-jose-json-web-signature-11" target="_blank">JWS Draft</a> for more information.</p>

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

### Memberships
{: #s5-api-memberships}

### Tokens
{: #s5-api-tokens}

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
| scope | list of granted scopes | strings separated by spaces |
| client_id | ... | string |
| sub | ... | string |
| aud | ... | string |
| token_type | ... | string |
| sub_groups | ... | string |
{: .request}

<hr/>

### Instances
{: #s5-api-instances}

### Services
{: #s5-api-services}

### Subscriptions
{: #s5-api-subscriptions}

Coming soon!