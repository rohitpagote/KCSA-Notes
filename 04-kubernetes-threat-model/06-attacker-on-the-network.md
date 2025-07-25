# Kubernetes Threat Model: Attacker on the Network

An attacker who gains access to the **network** (e.g., internal cluster network, corporate LAN, or cloud VPC) can perform various attacks such as sniffing traffic, spoofing services, or exploiting misconfigured components.

## 🎯 Objective
- **Intercept or manipulate traffic** between cluster components.
- **Exploit insecure network configurations** or services exposed unnecessarily.
- Perform **lateral movement** or data exfiltration.

## 🔍 Potential Attack Vectors

| Attack Type | Description |
|-------------|-------------|
| **Traffic Sniffing** | Capturing unencrypted data between components (e.g., etcd, kubelets). |
| **Man-in-the-Middle (MitM)** | Injecting malicious responses or altering requests. |
| **DNS Spoofing** | Redirecting service discovery to malicious endpoints. |
| **IP Spoofing** | Pretending to be another trusted node or service. |
| **ARP Poisoning** | Altering the ARP table to reroute traffic through attacker-controlled devices. |

## 🔐 Mitigation Strategies

| Mitigation | Description |
|------------|-------------|
| **Use TLS Everywhere** | Ensure encryption for API server, etcd, kubelet, and intra-cluster communication. |
| **mTLS (Mutual TLS)** | Strongly authenticate both server and client. |
| **Network Policies** | Restrict pod-to-pod and pod-to-service communication using Kubernetes NetworkPolicies or CNI plugins. |
| **Disable Unused Ports** | Do not expose insecure or unnecessary endpoints publicly. |
| **RBAC + Node Restriction** | Limit access based on node/service identity. |
| **VPC and Firewall Rules** | Isolate cluster and limit inbound/outbound access tightly. |
| **Monitoring and Logging** | Use Prometheus and Alertmanager to catch abnormal API usage and network spikes.  |

---

## 📌 Summary
- Enforce strict firewall rules around control-plane ports.
- Keep nodes and cluster components up to date.
- Apply NetworkPolicy for granular traffic control.
- Use MFA and RBAC to secure API access.
- Monitor API calls and network metrics to detect and respond rapidly.
