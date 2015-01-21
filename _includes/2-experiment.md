## Experiment
{: #ref-2}

### Preproduction sandbox
{: #ref-2-1}

The previous section (especially [Programming interface](#ref-1-3)) lists the production hosts of Ozwillo, all of which being subdomains of `ozwillo.com`. Ozwillo preproduction environment mirrors this setup under `ozwillo-preprod.eu` by providing the following hosts:

- `accounts.ozwillo-preprod.eu` for authentication pages and the Accounts API root;
- `kernel.ozwillo-preprod.eu` as the Kernel API root;
- `data.ozwillo-preprod.eu` as the the Datacore API root.
- `portal.ozwillo-preprod.eu` the portal for logged users;
- `www.ozwillo-preprod.eu` for information pages;

As of today, the preproduction fulfills two different objectives:

- as its name implies, the preproduction may serve a different (more recent) software version than production, showcasing its evolution in a future update;
- it's also the right place for providers to safely test and discover the APIs.

That's why providers are asked to implement provisioning and authentication on preproduction first. Once validated, their applications are declared to production.

### Authentication schemes
{: #ref-2-2}

This paragraph provides an overview of authentication and authorization schemes that occur between Ozwillo and provider APIs.

##### Recognize and trust a Kernel call

As you will [soon](#ref-3-2-1) see, a purchase act on the portal causes the Kernel to POST provisioning requests to a provider endpoint. Encoding the payload of the request and sending the resulting signature as a HTTP header allows the provider to recognize the Kernel as the request issuer.

Knowing this request intent is to create a new application instance, it includes a `client_id`/`client_secret` pair used in the following authentication schemes.

##### Calling the Kernel without an access_token
{: #ref-2-2--2}

Having an `access_token` means an end user has successfully [authenticated](#ref-4) to Ozwillo (within a given `client_id`). But there is a number of cases where the provider try to reach the Kernel before or outside an authenticated user context:

- precisely when [asking for](#ref-4-3-4) an `access_token` during the authentication flow;
- when answering the provisioning request described in the previous paragraph;
- when posting notifications or events.

In those cases, the provider servers must issue HTTP requests to the Kernel with [basic authentication](https://tools.ietf.org/html/rfc2617#section-2) using the `client_id`/`client_secret` pair as user-ID and password.

##### Calling the Kernel with an access_token

When the provider calls the Kernel with an `access_token`, it means the request is done on behalf a user. It helps the Kernel deciding if a particular operation is allowed depending on the user identity, [scopes]() and client_id associated with this `access_token`.

The corresponding resquests require [OAuth 2.0 Bearer](http://tools.ietf.org/html/rfc6750), meaning that the `access_token` is sent in an `Authorization: Bearer {access_token_value}` HTTP header.

### Recommendations
{: #ref-2-3}

##### HTTPS

Knowing that sensitive information may be exchanged between the Kernel and providers APIs (`client_id`, `access_token`), it is required that communication is done in HTTPS.

It may occur that your HTTPS configuration is not trusted by the Kernel and thus that some requests are not even sent. In this case, please use an SSL report tool to check your configuration or refer to the <a href="https://www.eff.org/https-everywhere/deploying-https" target="_blank">EFF's guide</a>.

##### Robustness

Following the <a href=" https://en.wikipedia.org/wiki/Robustness_principle" target="_blank">robustness principle</a> of Jon Postel, your API endpoints should be prepared to receive more data than expected and described in the documentation. It especially enables Ozwillo to add new functionality without breaking backward compatibility, thanks to new parameters or headers in HTTP requests or responses.