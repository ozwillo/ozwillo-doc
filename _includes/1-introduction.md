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
  <dt id="def-app-factory">App Factory</dt>
  <dd>A REST API (implemented by the provider) that Ozwillo can call to initiate the <a href="#ref-3">provisioning</a> of an application instance, after a purchase act has occured.</dd>   
</dl>

What may be confusing at first is that both *abstract* — not instantiated — applications and *tangible* — instantiated — services can be found in the store. From a user standpoint it makes sense: one does not worry about the technical implications behind a purchase act. But since the documentation reader cares, here is the difference:

- installing an application triggers the provisioning protocol, leading to software deployment on the provider side, and then to a declaration of the created instance to Ozwillo. It requires communication between Ozwillo and the provider servers;
- installing a service is simply the process of bookmarking the service URI as an icon on the user's desktop. This is instantaneous and does not require communication between Ozwillo and the provider servers.

### Visibility and access rights
{: #ref-1-4}

If an application or service is visible, it only means it is publicly referenced and shown under Ozwillo's store.

A `visible:false` (hidden) application may be under review or not be available at this time. A hidden service is typically targeted to certain users (depending on who purchased the instance): it is shown as a "service launcher icon" under one's desk, but not listed in the store.

The fact that a service is visible or not in the store is yet separated to how its access is managed, possibly being restricted to certain users. All of these scenarios are possible:

- a visible service whose access is restricted to given users (for example if the service owner wants to explicitely grant access to requesting users)
- a hidden service with public access (for example if the service owner does not want to be referenced under Ozwillo's store)
- more typically services that are either visible and public, or hidden and protected.

The [User Authentication](#ref-4) section shows how these authorization scenarios can be implemented.

### Documentation conventions
{: #ref-1-5}

HTTP requests and responses bodies (and data models) are described in tables. Within these tables the first column is dedicated to field or parameter names; when they are displayed in **boldface**, it means they are mandatory.

When you see curly brackets {} in a code sample, it means the content should be evaluated. Curly brackets typically contain variable names or describe an operation, for example:

<pre>
POST {instantiation_uri}
X-Hub-Signature: sha1={HMAC-SHA1 digest of payload}
</pre>
