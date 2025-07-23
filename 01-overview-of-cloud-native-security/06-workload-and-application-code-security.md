# Workload and Application Code Security

Secure your Kubernetes workloads and application code via runtime protection, secure coding, dependency checks, and visibility.

## 1. Workload Security (Container Runtime)  
- Apply **platform-level monitoring and runtime defense** (e.g., Sysdig Secure).
- Use **Kube-bench**, Falco to verify security benchmarks at runtime.
- Enforce security contexts and minimal container privileges.

## 2. Application Code Security  
- Secure code before it enters the runtime.

#### Prevent SQL Injection  
- ❌ Vulnerable:
  ```sql
  SELECT * FROM users WHERE username = '$user' AND password = '$pass';
  ```
- ✅ Use parameterized queries/prepared statements (e.g., SQLAlchemy text bindings). 
  ```ini
  # Example in Python with SQLAlchemy
  from sqlalchemy import text

  stmt = text("SELECT * FROM users WHERE username=:user AND password=:pass")
  result = engine.execute(stmt, {"user": user_input, "pass": password_input})
  ```

### Static Analysis Tools
- Automated scanners detect unsafe patterns like raw SQL concatenation before code merges into main:

| Tool      | Namespace                          | Key Feature                   |
|-----------|------------------------------------|------------------------------|
| SonarQube | Java, JavaScript, Python, and more | Highlights security hotspots |
| ReSharper | .NET/C#                            | Integrates into Visual Studio |
| Veracode  | Multiple                           | Cloud-based vulnerability scanning |
| Codacy    | JavaScript, Python, Ruby, Java     | Inline code review with CI plugins |

## 3. Scanning Third-Party Dependencies
- Regularly scan third-party libraries for CVEs (Common Vulnerabilities and Exposures).

#### Dependency Scanners

| Scanner                | Ecosystem                                           | Description                   |
|------------------------|--------------------------------------------------------|------------------------------|
| OWASP Dependency-Check | Java, .NET | Highlights security hotspots | Matches manifest files (pom.xml, packages.config) to CVE databases | 
| Snyk | .NET/C#         | JavaScript, Python, Go, Java              | Continuous monitoring with automatic pull requests | 
| GitHub Dependabot	     | Multiple                                  | Native GitHub alerts and automated dependency updates |

## 4. Runtime Security & Observability
- Protect against RCEs like Log4Shell using runtime WAFs: Datadog APM, AWS/Azure WAF. 
- Monitor syscall events, resource usage, and network flows.
- Tools: Sysdig Secure, app security telemetry platforms. 

#### Key Observability Features

| Capability           | Benefit                                            | 
|----------------------|----------------------------------------------------|
| System Call Tracing	 | Detect suspicious process events and file access   |
| Resource Metrics     | Identify CPU/memory spikes that may signal attacks |
| Network Monitoring   | Visualize container-to-container traffic flows     |

---

# ✅ Summary
| Area                      | Focus                           | Tools / Practices                                        |
| ------------------------- | ------------------------------- | -------------------------------------------------------- |
| **Workload Security**     | Runtime monitoring & compliance | Sysdig, Falco, Kube-bench, security contexts             |
| **Code Security**         | Safe coding & prevention        | Parameterized queries, static analysis (SonarQube, etc.) |
| **Dependency Scanning**   | Third-party library vetting     | OWASP DC, Snyk, Dependabot                               |
| **Runtime Observability** | Threat detection & anomaly logs | Syscall tracing, WAF, metrics/log correlation            |
