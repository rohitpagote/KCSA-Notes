# Kubernetes Threat Model: Access to Sensitive Data

Attackers gaining unauthorized access to **sensitive data** within a Kubernetes environment can lead to severe breaches, including credential theft, application compromise, and data exfiltration.

## 🎯 Targeted Sensitive Data

| Data Type           | Description |
|---------------------|-------------|
| **Secrets**         | Stored in Kubernetes (e.g., DB passwords, API tokens) |
| **ConfigMaps**      | May contain environment configurations or sensitive settings |
| **Pod Logs**        | Logs can include secrets, tokens, or personal data |
| **Volumes**         | Attached PVCs or ephemeral volumes may contain sensitive files |
| **Environment Variables** | Injected via deployment manifests or Helm charts |
| **Service Account Tokens** | Used by pods to access the API server |

## 🔍 Attack Vectors

| Vector | Description |
|--------|-------------|
| **Misconfigured RBAC** | Broad roles may allow unauthorized read access to secrets/configs |
| **Compromised Pod/Container** | Attacker inside a pod can read mounted secrets or env vars |
| **Unrestricted API Access** | APIs without proper authentication/authorization can leak data |
| **Insecure Workloads** | Logs and debug endpoints exposing sensitive info |
| **Shared Volumes** | Malicious pods reading data from shared volumes |

## 🧰 Mitigation Strategies

| Mitigation | Description |
|------------|-------------|
| **RBAC Minimization** | Apply the principle of least privilege to roles and bindings |
| **Encrypt Secrets at Rest** | Use Kubernetes secret encryption with KMS |
| **Enable Audit Logs** | Track who accessed secrets and when |
| **Avoid Secrets in Logs/Env** | Don’t store secrets in log files or as environment variables |
| **Restrict Pod Access** | Use PodSecurity policies and NetworkPolicies to isolate workloads |
| **Rotate Secrets Regularly** | Prevent long-lived exposure in case of leaks |
| **Disable Kubernetes Dashboard or Protect It** | Avoid exposing dashboards with broad access rights |
