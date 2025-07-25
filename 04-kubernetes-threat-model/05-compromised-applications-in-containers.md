# Kubernetes Threat Model: Compromised Applications in Containers

- When an application running inside a container is exploited, attackers can gain unauthorized access or misuse its capabilities. - This threat vector focuses on **application-level vulnerabilities**, not the container or node directly.

## 🎯 Objective
- Gain unauthorized access by exploiting **application vulnerabilities**.
- Use the app's permissions to **steal data**, move laterally, or escalate privileges.
- Deploy malware or persist access through the compromised app.

## 💣 Common Exploitation Techniques

| Technique | Description |
|----------|-------------|
| **Injection Attacks** | SQL, Command, or Code Injection to execute arbitrary commands. |
| **Remote Code Execution (RCE)** | Vulnerabilities in app code allowing execution of attacker-controlled scripts. |
| **Exposed Secrets** | Secrets hardcoded in code or stored insecurely (e.g., `.env` files). |
| **Misconfigured APIs** | Open or unauthenticated API endpoints. |
| **Directory Traversal** | Gain access to unintended files on the container's filesystem. |

## 🔐 Mitigation Strategies

| Mitigation                       | Description |
|----------------------------------|-------------|
| **Use Minimal Base Images**      | Reduce OS-level vulnerabilities with lightweight, well-maintained images (e.g., distroless).
| **Enforce Image Signing**        | Scan and sign images using tools like Notary or Cosign before deployment.
| **Rotate Secrets Regularly**     | Automate secret rotation and avoid long-lived credentials.
| **Apply Least-Privilege RBAC**   | Grant each service account only the permissions it needs.
| **Implement Network Policies**   | Segment pod communication and limit egress to the Kubernetes API server.
