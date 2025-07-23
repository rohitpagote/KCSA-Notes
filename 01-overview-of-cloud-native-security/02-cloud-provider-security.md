# Cloud Provider Security 

Cloud providers (AWS, Azure, GCP) supply multiple layers of infrastructure security—ranging from firewalls to advanced threat detection, WAFs, and container defenses. Below is an overview of these capabilities.

## 🔍 1. Threat Management & Response  
| Provider | Service                    | Description                                           |
|---------|-----------------------------|-------------------------------------------------------|
| AWS     | **GuardDuty**               | ML-driven threat detection for AWS accounts & workloads |
| Azure   | **Azure Sentinel**          | SIEM + SOAR for centralized alerting & incident response |
| GCP     | **Security Command Center** | Provides centralized dashboards for asset inventory, vulnerability scanning, alerting      |

- SIEM (Security Information and Event Management) 
- SOAR (Security Orchestration, Automation and Response)

## 🔥 2. Web Application Firewall (WAF)  
| Provider | WAF Feature Highlights                                           |
|---------|------------------------------------------------------------------|
| AWS     | AWS WAF (integration with CloudFront/ALB), Shield for DDoS protection |
| Azure   | Azure WAF + Application Gateway, OWASP rules, custom rules       |
| GCP     | Cloud Armor, DDoS defense, geo-based & custom policies           |

- Designed to protect against OWASP Top 10 attacks and DDoS

## 📦 3. Container Security  
| Provider | Service & Features                                                                 |
|---------|--------------------------------------------------------------------------------------|
| AWS     | EKS + Bottlerocket (special OS, specific to EKS); kube-bench, IAM roles for service accounts |
| Azure   | AKS with control-plane hardening, Azure Policy integration, image scanning         |
| GCP     | GKE private clusters, Anthos policy via OPA (Open Policy Agent), Binary Authorization |

- Provides built-in runtime protection, compliance checks, and policy enforcement

## 🤝 4. Shared Responsibility Model  
- **Cloud provider**: Secures infrastructure (“of” the cloud).  
- **User**: Secures workloads (“in” the cloud) — OS, K8s config, applications.  

---

# ✅ Summary 
1. Activate firewalls to block unauthorized access and secure systems.
2. Cloud providers offers diverse security tools for protecting data.
3. Shared responsibility model divides security tasks between users and providers.
