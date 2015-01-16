## Provisioning
{: #ref-3}

This section specifies the provisioning protocol triggered between Ozwillo and the provider APIs when a [purchase act](#def-purchase-act) occurs on the portal. The outcome of a successful provisioning is an application instance that is both:

- allocated and reachable on the provider server;
- declared with appropriate settings, especially regarding access rights, on Ozwillo.

The creation of an application instance on the provider side may imply configuration changes or new resources allocation. These implementation details are left unconstrained: the protocol focuses only on API communication. It means existing deployment operations does not have to be affected and should rather be exposed through this new protocol.


### Prerequesite
{: #ref-3-1}

declared in the catalog, send a mail to contact@ozwillo.com