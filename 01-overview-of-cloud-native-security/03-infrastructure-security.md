# Infrastructure Security

Protect cloud-native compute, network, and storage. Focus on network segmentation, server hardening, and secure secrets handling. 

## 🧩 Stage 1: Network Segmentation & API Exposure
| Vulnerability	 | Risk                    | Mitigation                                           |
|---------|--------------------------|-------------------------------------------------------|
| Shared IP hosting multiple domains | Compromise of one app exposes all apps | Use VPCs/Subnets or separate servers |
| Public Kubernetes API endpoint   | Unrestricted API discovery and access | Remove public IP, enforce VPN or private endpoints |

- **Fixes**:
  - Isolate workloads in separate VPCs/subnets or servers.
  - Remove public K8s API IP; enforce VPN or private endpoints.
  - Implement firewall rules and ACLs.

## 🐳 Stage 2: Securing Docker Daemon Access
- Default Docker daemon port `2375` exposed → remote control without TLS.
- **Fixes**:
  - Block port `2375` using iptables/ufw or cloud firewalls.
  - Enable Docker TLS (`--tlsverify`).
  - Use K8s NetworkPolicies to limit Pod‑to‑Pod/Host traffic.

# 🎛️ Stage 3: Least Privilege & RBAC Enforcement
- Privileged containers + open Dashboard = full cluster compromise.
- **Fixes**:
  - Enforce non-root execution and drop container capabilities.
  - Secure Kubernetes Dashboard with RBAC and auth.
  - Rotate service account tokens; scope them properly. 

## 🔐 Stage 4: Secure Secrets & etcd
- Secrets in plaintext env vars → stolen by attackers.
- **Fixes**:
  - Use K8s Secrets encrypted at rest.
  - Enable etcd encryption providers.
  - Secure etcd with TLS for clients and peers.
  - Apply strict RBAC to etcd access.
  - Audit backups and provider-managed etcd. 

---

# ✅ Summary
1. Segment networks and block public API access.
2. Lock down Docker daemon (port 2375) and enforce TLS/firewalls.
3. Apply least privilege, secure dashboard with RBAC.
4. Encrypt secrets in K8s, secure etcd with TLS/RBAC.