# Kubernetes Threat Model: Denial of Service (DoS)

**Denial of Service (DoS)** attacks in Kubernetes aim to exhaust system resources—making the cluster or its workloads **unavailable to legitimate users**.

## 💥 What is a DoS Attack?
A **DoS attack** attempts to:
- Consume CPU, memory, disk, or network resources.
- Overwhelm Kubernetes components (like the API server).
- Cause application or infrastructure failures.

## 🔍 DoS Attack Vectors in Kubernetes

| Attack Vector | Description |
|---------------|-------------|
| **Pod Resource Exhaustion** | Running pods that consume **all available CPU or memory**. |
| **No Resource Limits** | Pods without `resources.limits` can consume unlimited resources. |
| **API Server Overload** | Sending too many requests to the API server (e.g., with `kubectl get pods -w`). |
| **Log Flooding** | Generating excessive logs to consume disk and I/O. |
| **Event Spamming** | Repeatedly creating/deleting resources to spam the event system. |
| **CrashLoopBackOff Loops** | Misconfigured pods crashing and restarting endlessly, stressing the system. |

## 🔎 Detecting DoS Behavior
- Monitor for:
    - Nodes running out of memory or CPU.
    - High container restarts or CrashLoopBackOffs.
    - API server latency or request spikes.
    - Disk usage spikes from excessive logs or events.
- Tools to use:
    - Prometheus + Grafana
    - Kube-state-metrics
    - Log aggregation (e.g., Fluentd, Loki)

## 🛡️ Mitigating DoS Attacks

| Mitigation Step                      | Description                                                          |
| ------------------------------------ | -------------------------------------------------------------------- |
| **Set Resource Requests & Limits**   | Prevent pods from consuming all resources.                           |
| **Least-Privilege Service Accounts** | Define RBAC roles with minimal privileges                            |
| **Rate-limit API Requests**          | Prevent abuse via throttling (e.g., using API server flags).         |
| **Quota Management**                 | Apply **ResourceQuotas** and **LimitRanges** at the namespace level. |
| **Monitoring and Alerts**            | Use Prometheus and Grafana to detect unusual resource spikes         |
| **Use Network Policies**             | Block unauthorized traffic that may generate load.                   |

## 📌 Summary
To defend against Kubernetes DoS attacks:
- Enforce resource quotas and limits.
- Apply least-privilege RBAC for service accounts.
- Implement network policies and firewall rules.
- Continuously monitor and alert on anomalies.
