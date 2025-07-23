# Artifact Repository and Image Security

Secure your container supply chain - from image creation to runtime - to prevent vulnerabilities and tampering.

## 1. Risks of Untrusted Base Images
- `latest` tag via Docker Hub = potential CVEs, instability (Relying on the `latest` tag does not ensure up-to-date security patches).
- Always verify image origin, patch status, and maintainers.

## 2. Integrating Vulnerability Scanning
- Integrate scanners into CI/CD:

```bash
# Scan with Trivy
trivy image --severity HIGH,CRITICAL team-a/crm:stable

# Scan with Clair via clair-scanner
clair-scanner --ip $(hostname -I | awk '{print $1}') team-a/crm:stable
```

| Scanner | Description                                  | Command Example                        |
|---------|----------------------------------------------|----------------------------------------|
| Trivy   | Lightweight, fast vulnerability scanner      | `trivy image <image>`                  |
| Clair   | Static analysis of vulnerabilities in images | `clair-scanner --ip <host-ip> <image>` |

## 3. Adopting Minimal Official Base Images
- Prefer minimal images (e.g., Alpine, Ubuntu).
- Reduces attack surface and ensures timely patching.

## 4. Build Artifacts & Repositories
- Build artifacts include compiled binaries, JAR/WAR files, logs, reports, and especially container images.
- Use centralized artifact repositories for security & control.

## 5. Storing Container Images
- While Docker Hub is popular for hosting images, it has limited access controls and no built-in vulnerability scanning.

| Repository        | Access Control | Scanning   | Image Signing | 
|-------------------|----------------|------------|---------------|
| Docker Hub        | Basic          | No         | No            |
| Nexus Repository  | Fine-grained   | Via add-on | Limited       |
| GitHub Packages   | Fine-grained   | Yes        | Yes           |
| JFrog Artifactory | Fine-grained   | Yes        | Yes           |

---

# ✅ Next Steps
1. Integrate automated scans in your CI/CD pipeline.
2. Standardize on minimal, official base images.
3. Use a robust artifact repository with access controls and signing.
4. Continuously monitor and update images to address new vulnerabilities.