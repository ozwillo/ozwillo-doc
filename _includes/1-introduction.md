## Introduction
{: #ref-1}

### User interface and portal features
{: #ref-1-1}

From a user standpoint, the Ozwillo portal acts similarly to a mobile operating system, since it's possible to:

- find and install applications from Ozwillo's store
- launch installed services (see the difference between applications and services [below](#ref-1-3))
- manage one's personal and application settings

The main user interface components are:

- information pages served under `www.ozwillo.com`
- the portal (as defined above) and its store, served under `portal.ozwillo.com`
- authentication-related pages served under `accounts.ozwillo.com`

Except from the store, the portal pages are reserved to registered and logged users. They especially give access to:

- one's desk, made of shortcut icons to launch services;
- one's network, to create and manage organizations and links within organizations. Thanks to this, it is possible for someone to install applications on behalf organizations, in addition to a personal use, if one is an admin of this organization;
- one's application settings, where one can give others access to a given service, typically an organization admin authorizing other organization members.

### Programming interface
{: #ref-1-2}

From a provider standpoint, adapting to Ozwillo means:

1. include the application deployment process within the [provisioning protocol](#ref-3) described below. This is dealt with requests exchanged between the Kernel REST API and provider endpoints (typically over a REST API too)
2. if user authentication is needed, [delegate it](#ref-4) to Ozwillo OpenID Connect Identity Provider
3. create, update and consume linked and shared data through the Datacore REST API

The Datacore API and its documentation are stabilizing. For the moment this document focus on the 2 first and necessary steps regarding Ozwillo integration: provisioning and authentication.
{: .focus .soft}

In other words, Ozwillo programming interface is made of:

- the Kernel API available under `kernel.ozwillo.com`
- the Accounts API and web pages (single sign-in, single sign-out, forgotten password...) available under `accounts.ozwillo.com`
- the Datacore API available under `data.ozwillo.com`

**NB**: Ozwillo APIs access is over HTTPS, providers endpoints must be too.
{: .focus .important}

### Terminology
{: #ref-1-3}

We make no asumption regarding what words final users will use between *applications* or *services*. It means that one could think "I am launching this app" when another would say "I am accessing this service".

But when it comes to Ozwillo's APIs and thus to this documentation, they have a different and precise meaning. From the general to the particular:

<dl>
  <dt>Application</dt>
  <dd>an abstract application, not directly usable, declared in the catalog and visible in Ozwillo's store, that can be the object of an instantiation. It's the product you sell;</dd>
  <dt id="def-application-instance">Application instance</dt>
  <dd>or **instance** for short, a runnable copy of an application, created within the context of an identified authority (which may be an individual or an organization) after a successful provisioning;</dd>
  <dt>Service</dt>
  <dd>an "endpoint" of an application instance, addressable through a URL (the service URI). Services are declared in the catalog and may or may not be visible in Ozwillo's store. An application instance is made of one or more services.</dd>
  <dt id="def-desk-shortcuts">Desk shortcuts</dt>
  <dd>The icons shown in a user's desk (under the Ozwillo portal) are links to services URIs.</dd>
</dl>

Additional definitions to complement the picture:

<dl>
  <dt id="def-catalog">(Ozwillo's) catalog</dt>
  <dd>a database of all applications, instances and services. It is used internally by the Portal to display a user’s desktop, to browse the app store, etc.;</dd>
  <dt>(Ozwillo's) store</dt>
  <dd>a system that allows users to: find services that are publicly available and add them to their desktops, or find applications that are instantiable through provisioning. The store is the visible surface of the catalog;</dd>
  <dt id="def-store-entry">Store entry</dt>
  <dd>A visible application or a visible service (but not an application instance, whose visible surface is always a service);</dd>
  <dt id="def-purchase-act">Purchase act</dt>
  <dd>The act of requesting the installation of a story entry (charged or free), done by a portal end user;</dd>
  <dt id="def-purchaser">Purchaser</dt>
  <dd>A user that performed a purchase act;</dd>
  <dt id="def-app-factory">App factory</dt>
  <dd>A REST API (implemented by the provider) that Ozwillo can call to initiate the <a href="#ref-3">provisioning</a> of an application instance, after a purchase act has occured.</dd>   
</dl>

What may be confusing at first is that both *abstract* — not instantiated — applications and *tangible* — instantiated — services can be found in the store. From a user standpoint it makes sense: one does not worry about the technical implications behind a purchase act. But since the documentation reader cares, here is the difference:

- installing an application triggers the provisioning protocol, leading to software deployment on the provider side, and then to a declaration of the created instance to Ozwillo. It requires communication between Ozwillo and the provider servers;
- installing a service is simply the process of bookmarking the service URI as an icon on the user's desktop. This is instantaneous and does not require communication between Ozwillo and the provider servers.

### Visibility vs restricted access
{: #ref-1-4}

If an application or service is visible, it only means it is publicly referenced and shown under Ozwillo's store.

A `visible:false` (hidden) application may be under review or not be available at this time. A hidden service is typically targeted to certain users (depending on who purchased the instance): it is shown as a "service launcher icon" under one's desk, but not listed in the store.

The fact that a service is visible or not in the store is yet separated to how its access is managed, possibly being restricted to certain users. All of these scenarios are possible:

- a visible service whose access is restricted to given users (for example if the service owner wants to explicitely grant access to requesting users)
- a hidden service with public access (for example if the service owner does not want to be referenced under Ozwillo's store)
- more typically services that are either visible and public, or hidden and protected.

The [Provisioning](#ref-3) section shows how these settings are set.

### Authorized users
{: #ref-1-5}

### `app_admin` vs `app_user`
{: #ref-1-5-1}

The case for restricted services is that only specific users are allowed to access and interact with them. By default, the user that purchased the instance is an `app_admin`, meaning that s/he has the right to add new `app_admin` or `app_user` users.

Both `app_admin` and `app_user` are considered authorized users of the instance, but an `app_user` can not add other authorized users. These rules (who can add who) are internal to Ozwillo and frame the organization management under the portal.

As a provider, you will also get this info (is the user an `app_admin` or `app_user`) and are free to give it whatever meaning best suits your application behavior. Yet, on application instances side, it's typical that `app_admin` maps the role with the highest privileges and is allowed to configure the roles of others (`app_user`). Then `app_user` may hold a variety of different user types from the application point of view. Since Ozwillo can not know all business-specific roles of all applications, you can define a finer granularity of users under the global `app_user` concept linked to Ozwillo.

To sum-up, it is typical that on the application side:

- `app_admin` characterises users with the highest privileges;
- `app_user` is generic and may apply to different business roles (for instance: moderator, editor...).

Now, when your services are accessed by internet users, how do you know they are indeed authorized users (`app_admin` or `app_user`)? This will be answered in depth in the [User Authentication](#ref-4) section, but let's give a glimpse of it, it will also helps introduce the *scope* concept.

### Authorization in brief
{: #ref-1-5-2}

Let's say an internet user arrives at one of the URIs corresponding to the protected area of your service, typically the service URI entrypoint.

Without Ozwillo, you would check that you know this user (existing session) and if not trigger authentication to check if s/he can identify as an authorized user.

With Ozwillo, you delegate authentication and authorization: Ozwillo is both an identity provider and an authorization server. It means that without a valid session on your server, you redirect the user to Ozwillo authentication page, passing it interesting parameters like:

- a `client_id` that helps identify the application instance in which the service resides;
- a `scope` list that will be explained in the next paragraph;
- and others that will be detailed in the [User Authentication](#ref-1-4) section.

So let's focus on the `client_id`: thanks to it, Ozwillo knows what application instance the user is trying to access. Ozwillo then checks if s/he is indeed a valid `app_admin` or `app_user` of this instance, and now there are two cases depending on the service configuration:

- if the service is `restricted:true` Ozwillo authentication will only succeedd for an `app_admin` or `app_user`: you can rely on it;
- if the service is `restricted:false` Ozwillo authentication will succeed even if the user is neither an `app_admin` nor a `app_user`, but the service will be notified of it.

If the authentication is successful, your service will be given at the end of the [authorization code flow](#ref-4-3) an `access_token` that can be seen as a "session id" from Ozwillo point of view. Now you have an `access_token`, you can create a session linking the user and this `access_token`, so that authentifying this user won't be necessary on further pages access. For security measures, the `access_token` must not be leaked on the client side.

In short, you condition the creation of a session on your server to the retrieval of an `access_token` from Ozwillo.

### Scopes
{: #ref-1-5-3}

First of all, Ozwillo as an authentication and authorization solution is an implementation of <a href="http://openid.net/connect/" target="_blank">OpenID Connect</a> Authorization Server, and as such uses scopes introduced by the protocol to specify access privileges.

For instance and if we focus on scopes <a href="http://openid.net/specs/openid-connect-basic-1_0.html#Scopes" target="_blank">introduced</a> by OpenID Connect, the provider may ask for an `access_token` with the `email` scope. When authenticating, users are prompted to know if they want to share their email address with this instance. As a result, the `access_token` created are associated to this scope (if granted by user) on Ozwillo side.

When the instance wants to indeed access the email through one of Ozwillo API endpoints, it will send an HTTP request with this `access_token`, and Ozwillo is then able to decide if the operation is permitted or not.

As you will see later, you can ask for scopes at several occasions:

- during the application instance installation (users will globally accept or deny it for their future usage of the application instance);
- when accessing a service (prompt during authentication) and for the lifetime of the `access_token`;
- on a per action basis.

When a claimed scope is refused by users, the instance will always be able to ask for it again later, and explain a given operation won't be possible until they accept it.

Scopes are typically used to access private profile information (see those inherent in <a href="http://openid.net/specs/openid-connect-basic-1_0.html#Scopes" target="_blank">OpenID Connect</a>, like email, address or phone). But we will see that this mechanism is flexible enough to use additional scopes defined by application instances during [provisioning](#ref-3-2-3-scope).

### Documentation conventions
{: #ref-1-6}

HTTP requests and responses bodies (and data models) are described in tables. Within these tables the first column is dedicated to field or parameter names; when they are displayed in **boldface**, it means they are mandatory.

When you see curly brackets {} in a code sample, it means the content should be evaluated. Curly brackets typically contain variable names or describe an operation, for example:

<pre>
POST {instantiation_uri}
X-Hub-Signature: sha1={HMAC-SHA1 digest of payload}
</pre>
