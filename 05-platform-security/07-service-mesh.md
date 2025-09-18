# Service Mesh

- A Service Mesh is an infrastructure layer for managing service-to-service communication in microservices architecture.
- It offloads networking, security, and observability logic from service code to infrastructure.

## 🏗 Architecture Overview

- Each microservice instance has a **sidecar proxy** injected alongside it -> forms the **data plane**.  
- A **control plane** centrally configures these proxies: routing, policies, telemetry.  
- Developers focus on business logic; networking/security concerns are handled by the mesh.  

### Key benefits:
- **Dynamic Traffic Routing**: Canary releases, blue/green deployments, circuit breaking, retries
- **Mutual TLS (mTLS)**: Automatic encryption and authentication of service calls
- **Observability**: End-to-end metrics, logs, and distributed tracing
- **Service Discovery**: Automatic registration and lookup of service instances


## ⚙ Core Responsibilities & Features of Service Mesh

| Capability          | What it Does     | Examples / Tools / Configs        |
|---------------------|----------------------------------------------|--------------------------------------------------------|
| Service Discovery   | Keeps registry of healthy service instances; dynamic lookup of endpoints    | Envoy, Consul Catalog |
| Health Checking     | Periodically checks service instances; removes unhealthy ones from routing | HTTP / gRPC probes; custom health checks |
| Load Balancing      | Distributes traffic (round-robin, least connections, locality, etc.)        | Envoy LB algorithms |
| Security (mTLS)     | Encrypts + authenticates all inter-service traffic                | Istio PeerAuthentication, Linkerd identity service |
| Traffic Management  | Enables retries, timeouts, circuit breakers, canary/blue-green releases | VirtualService in Istio, ServiceProfile in Linkerd |
| Observability       | Logs, metrics, distributed tracing across services for better visibility    | Prometheus, Jaeger, Grafana |


### ✅ Benefits Summary

- **Automatic encryption & authentication** between services (no code changes).  
- **Better reliability**: retries, circuit breakers, health checks.  
- **Safe deployments**: traffic splitting & gradual rollouts (canary / blue-green).  
- **Improved visibility**: full service-to-service telemetry, logs & tracing.  
