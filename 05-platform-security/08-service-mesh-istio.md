# Service Mesh Istio

## What is Istio
Istio is a free, open source service mesh that secures, connects, and observes microservices. It integrates seamlessly with Kubernetes and virtual machine-based workloads to provide:
- Fine-grained traffic control and routing
- Automatic mutual TLS for service identity and encryption
- Telemetry collection and distributed tracing
- Policy enforcement and rate limiting

## Istio Architecture  

| Plane        | Role                                                                 |
|---------------|----------------------------------------------------------------------|
| **Control Plane (Istiod)** | Handles service discovery, policy configuration, certificate issuance/rotation, and validation & distribution of configuration |
| **Data Plane (Envoy Sidecars)** | Runs alongside each application workload; handles routing, TLS enforcement, telemetry, retries/failover |

## Core Components of Istio
- **Istiod**  
  - Unified control-plane binary (previously split into Pilot, Citadel, Galley).
  - Responsible for certificate management (mTLS), configuration validation & distribution.

- **Envoy Sidecar Proxy**  
  - Injected into workloads (pods) to intercept inbound & outbound traffic. 
  - Applies routing rules, retries, failover, secure comms, metrics/telemetry.

- **Istio Agent**  
  - Bootstraps the Envoy proxy. 
  - Retrieves x.509 certificates, streams configuration (via SDS/CDS), monitors proxy health.

## ✅ Summary  

| Component        | Plane         | Key Responsibility                                      |
|------------------|----------------|---------------------------------------------------------|
| Istiod           | Control Plane  | Certs, policy, service discovery, configuration         |
| Envoy Sidecar    | Data Plane     | Traffic routing, telemetry, enforcing mTLS & policies   |
| Istio Agent      | Data Plane     | Proxy bootstrap, config/cert delivery, health monitoring|
