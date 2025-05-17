# Annotations

Annotations can be added on Kubernetes resources to customize external-dns behavior. Part of these annotations serve to alter the way external-dns reads data from the source, others to configure the providers

## Common annotations

## Source-oriented annotations

| Name                                                                                                              | Values                                    | Default           | Supported Source                                                                               | Supported Providers                                                                                                   |
| ----------------------------------------------------------------------------------------------------------------- | ----------------------------------------- | ----------------- | ---------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------- |
| [external-dns.alpha.kubernetes.io/access](#external-dnsalphakubernetesioaccess)                                   | `private` \| `public`                     | N/A               | Service (NodePort)                                                                             | all ? (getAccessFromAnnotations)                                                                                      |
| [external-dns.alpha.kubernetes.io/alias](#external-dnsalphakubernetesioalias)                                     | bool                                      | `false`           | ? all ?                                                                                        | AWS hasAliasFromAnnotations                                                                                           |
| [external-dns.alpha.kubernetes.io/controller](#external-dnsalphakubernetesiocontroller)                           | string                                    | `dns-controller ` | Contour, Gateway, Ingress, Istio Gateway, Istio Virtual Service, Note, Openshift, Service, Ski | all ? controllerAnnotationKey                                                                                         |
| [external-dns.alpha.kubernetes.io/endpoints-type](#external-dnsalphakubernetesioendpoints-type)                   | `NodeExternalIP` \| `HostIP`              | N/A               | Service (Headless)                                                                             | all                                                                                                                   |
| [external-dns.alpha.kubernetes.io/hostname](#external-dnsalphakubernetesiohostname)                               | stringList                                | N/A               | Contour, Gateway, Ingress, Istio, Pod, Service, Skipper                                        | all                                                                                                                   |
| [external-dns.alpha.kubernetes.io/ingress-hostname-source](#external-dnsalphakubernetesioingress-hostname-source) | `defined-hosts-only` \| `annotation-only` | ?                 | Ingress                                                                                        | all                                                                                                                   |
| [external-dns.alpha.kubernetes.io/internal-hostname](#external-dnsalphakubernetesiointernal-hostname)             | string                                    | ?                 | Pod, Service[^1][^2]                                                                           | all                                                                                                                   |
| [external-dns.alpha.kubernetes.io/set-identifier](#external-dnsalphakubernetesioset-identifier)                   | ?                                         | ?                 | all                                                                                            | AWS                                                                                                                   |
| [external-dns.alpha.kubernetes.io/target](#external-dnsalphakubernetesiotarget)                                   | string                                    | ?                 | ?                                                                                              | Ambassador, Contour, F5, Gateway[^4], Gloo, Ingress, Istio, Kong, Node, OpenShift, Pod, Service[^3], Skipper, Traefik |
| [external-dns.alpha.kubernetes.io/ttl](#external-dnsalphakubernetesiottl)                                         | int ?                                     | ?                 | ?                                                                                              | Ambassador, Contour, F5, Gateway, Gloo[^5], Ingress, Istio, Kong, Node, OpenShift, Service, Skipper, Traefik          |

[^3]: Also supported on `Pods` referenced from a headless `Service`'s `Endpoints`.
[^4]: The annotation must be on the `Gateway`.
[^5]: The annotation must be on the listener's `VirtualService`.

## Provider-specific annotations

Some providers define their own annotations. Cloud-specific annotations have keys prefixed as follows:
Supported by Source: Ambassador, Contour, Gateway, Gloo[^5], Ingress, Istio, Kong, OpenShift, Service, Skipper, Traefik

| Cloud      | Annotation prefix                              |
| ---------- | ---------------------------------------------- |
| AWS        | `external-dns.alpha.kubernetes.io/aws-`        |
| CloudFlare | `external-dns.alpha.kubernetes.io/cloudflare-` |
| IBM Cloud  | `external-dns.alpha.kubernetes.io/ibmcloud-`   |
| Scaleway   | `external-dns.alpha.kubernetes.io/scw-`        |

## external-dns.alpha.kubernetes.io/access

Specifies which set of node IP addresses to use for a `Service` of type `NodePort`.

If the value is `public`, use the Nodes' addresses of type `ExternalIP`, plus IPv6 addresses of type `InternalIP`.

If the value is `private`, use the Nodes' addresses of type `InternalIP`.

If the annotation is not present and there is at least one address of type `ExternalIP`,
behave as if the value were `public`, otherwise behave as if the value were `private`.

### external-dns.alpha.kubernetes.io/alias

If the value of this annotation is `true`, specifies that CNAME records generated by the
resource should instead be alias records.

!!! info

    This annotation is only relevant if the `--aws-prefer-cname` flag is specified.

## external-dns.alpha.kubernetes.io/controller

If this annotation exists and has a value other than `dns-controller` then the source ignores the resource.

## external-dns.alpha.kubernetes.io/endpoints-type

Specifies which set of addresses to use for a headless `Service`.

If the value is `NodeExternalIP`, use each relevant `Pod`'s `Node`'s address of type `ExternalIP`
plus each IPv6 address of type `InternalIP`.

If the value is `HostIP` or the `--publish-host-ip` flag is specified, use
each relevant `Pod`'s `Status.HostIP`.

If unset, use the `IP` of each of the `Service`'s `Endpoints`'s `Addresses`.

## external-dns.alpha.kubernetes.io/hostname

Specifies the domain for the resource's DNS records.

Multiple hostnames can be specified through a comma-separated list, e.g.
`svc.mydomain1.com,svc.mydomain2.com`.

For `Pods`, uses the `Pod`'s `Status.PodIP`, unless they are `hostNetwork: true` in which case the NodeExternalIP is used for IPv4 and NodeInternalIP for IPv6.

## external-dns.alpha.kubernetes.io/ingress-hostname-source

Specifies where to get the domain for an `Ingress` resource.

If the value is `defined-hosts-only`, use only the domains from the `Ingress` spec.

If the value is `annotation-only`, use only the domains from the `Ingress` annotations.

If unset, use the domains from both the spec and annotations.

## external-dns.alpha.kubernetes.io/internal-hostname

Specifies the domain for the resource's DNS records that are for use from internal networks.

For `Services` of type `LoadBalancer`, uses the `Service`'s `ClusterIP`.

For `Pods`, uses the `Pod`'s `Status.PodIP`, unless they are `hostNetwork: true` in which case the NodeExternalIP is used for IPv4 and NodeInternalIP for IPv6.

Unless the `--ignore-hostname-annotation` flag is specified.

[^2]: Only behaves differently than `hostname` for `Service`s of type `ClusterIP` or `LoadBalancer`.

### external-dns.alpha.kubernetes.io/set-identifier

Note: only implemented by AWS

Specifies the set identifier for DNS records generated by the resource.

A set identifier differentiates among multiple DNS record sets that have the same combination of domain and type.
Which record set or sets are returned to queries is then determined by the configured routing policy.

## external-dns.alpha.kubernetes.io/target

Specifies a comma-separated list of values to override the resource's DNS record targets (RDATA).

Targets that parse as IPv4 addresses are published as A records and
targets that parse as IPv6 addresses are published as AAAA records. All other targets
are published as CNAME records.

## external-dns.alpha.kubernetes.io/ttl

Specifies the TTL (time to live) for the resource's DNS records.

The value may be specified as either a duration or an integer number of seconds.
It must be between 1 and 2,147,483,647 seconds.
