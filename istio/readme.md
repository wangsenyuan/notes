1. Envoy, Mixer & Pilot
  1. Envoy is a high-performance proxy developed in C++ to mediate all inbound and outbound traffic for all services in the service mesh. Envoy is deployed as a sidecar to the relevant service in the same Kubernetes pod. 
  2.  Mixer enforces access control and usage policies across the service mesh, and collects telemetry data from the Envoy proxy and other services
  3.  Pilot provides service discovery for the Envoy sidecars, traffic management capabilities for intelligent routing (e.g., A/B tests, canary deployments, etc.), and resiliency (timeouts, retries, circuit breakers, etc.).Pilot converts high level routing rules that control traffic behavior into Envoy-specific configurations, and propagates them to the sidecars at runtime.
2. Rule Configuration
  There are four traffic management configuration resources in Istio: VirtualService, DestinationRule, ServiceEntry, and Gateway:
  1. A VirtualService defines the rules that control how requests for a service are routed within an Istio service mesh.
  2. A DestinationRule configures the set of policies to be applied to a request after VirtualService routing has occurred.
  3. A ServiceEntry is commonly used to enable requests to services outside of an Istio service mesh.
  4. A Gateway configures a load balancer for HTTP/TCP traffic, most commonly operating at the edge of the mesh to enable ingress traffic for an application.
3. problems not clear yet
  1. 部署了某个service， foo.bar.com，（存在于pod中），istio如何配置，如何将DestinationRule和该Pod关联？通过label?
  2. 部署同一个service的不同版本，在k8s中，需要定义不同的deployment？service?
  3. 在使用istio的时候, k8s service 是否不再需要了？