# Linkerd

Linkerd is a service mesh.

##  Some key features of Linkerd service mesh

- Automatic mTLS. Linkerd automatically enables mutual Transport Layer Security (TLS) for all communication between meshed applications.
- Automatic Proxy Injection. Linkerd will automatically inject the data plane proxy into your pods based annotations.
- Traffic Split (canaries, blue/green deploys). Linkerd can dynamically send a portion of traffic to different services.
- Multi-cluster communication. Linkerd can transparently and securely connect services that are running in different clusters.
  Non-Kubernetes workloads (mesh expansion)
- Linkerd features mesh expansion, or the ability to add non-Kubernetes workloads to your service mesh by deploying the Linkerd proxy to the remote machine and connecting it back to the Linkerd control plane within the mesh.

##  Deep dive into two important Linkerd features

###  Automatic mTLS

By default, Linkerd automatically enables mutually-authenticated Transport Layer Security (mTLS) for all TCP traffic between meshed pods.

####  What is mTLS?

mTLS is simply “regular TLS” with one extra stipulation: the client is also authenticated.

#### How does it work?

The Linkerd control plane contains a certificate authority (CA) called identity. This CA issues TLS certificates to each Linkerd data plane proxy. Each certificate is bound to the Kubernetes ServiceAccount identity of the containing pod. These TLS certificates expire after 24 hours and are automatically rotated. The proxies use these certificates to encrypt and authenticate TCP traffic to other proxies.
On the control plane side, Linkerd maintains a set of credentials in the cluster: a trust anchor, and an issuer certificate and private key.
On the data plane side, each proxy is passed the trust anchor in an environment variable.
The proxy connects to the control plane’s identity component, validating the connection to identity with the trust anchor, and issues a certificate signing request (CSR). The CSR contains an initial certificate with identity set to the pod’s Kubernetes ServiceAccount, and the actual service account token, so that identity can validate that the CSR is valid. After validation, the signed trust bundle is returned to the proxy, which can use it as both a client and server certificate. These certificates are scoped to 24 hours and dynamically refreshed using the same mechanism.

### Telemetry and Monitoring

One of Linkerd’s most powerful features is its extensive set of tooling around observability—the measuring and reporting of observed behavior in meshed applications. While Linkerd doesn’t have insight directly into the internals of service code, it has tremendous insight into the external behavior of service code.

####  Metrics

- Success rate: This is the percentage of successful requests during a time window (1 minute by default).
- Traffic: This gives an overview of how much demand is placed on the service/route.
- Latencies: Times taken to service requests per service/route are split into 50th, 95th and 99th percentiles. Lower percentiles give you an overview of the average performance of the system, while tail percentiles help catch outlier behavior.
