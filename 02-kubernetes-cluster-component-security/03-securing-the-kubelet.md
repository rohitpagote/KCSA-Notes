# Securing the Kubelet

In Kubernetes, the kubelet acts as the "captain" on each worker node. It:
- Registers the node with the control plane.
- Starts and stops containers per Pod specs.
- Monitors Pod and container health, reporting back to the API Server.

## 1. Role & Installation of the Kubelet

### Key Responsibilities
- Node Registration
- Pod Lifecycle Management
- Health Reporting

### Manual Installation
- Download the kubelet binary and set up a systemd unit:
```bash
wget https://storage.googleapis.com/kubernetes-release/release/v1.20.0/bin/linux/amd64/kubelet \
  -O /usr/local/bin/kubelet && chmod +x /usr/local/bin/kubelet
```

```ini
# /etc/systemd/system/kubelet.service
[Unit]
Description=Kubelet Service
After=network.target

[Service]
ExecStart=/usr/local/bin/kubelet \
  --container-runtime=remote \
  --image-pull-progress-deadline=2m \
  --kubeconfig=/var/lib/kubelet/kubeconfig \
  --network-plugin=cni \
  --register-node=true \
  --cluster-domain=cluster.local \
  --cluster-dns=10.96.0.10 \
  --v=2
Restart=always
RestartSec=5

[Install]
WantedBy=multi-user.target
```

> [!NOTE] 
> With kubeadm (v1.10+), most flags migrate into /var/lib/kubelet/config.yaml and are maintained automatically during kubeadm join.

### Dedicated Config File
```yml
# /var/lib/kubelet/config.yaml
apiVersion: kubelet.config.k8s.io/v1beta1
kind: KubeletConfiguration
clusterDomain: cluster.local
clusterDNS:
  - 10.96.0.10
fileCheckFrequency: 0s
httpCheckFrequency: 0s
syncFrequency: 0s
healthzPort: 10248
```
- Add `--config=/var/lib/kubelet/config.yaml` to the service's ExecStart. 
- Command-line flags will always override the YAML settings.

## 2. Inspecting the Active Configuration
On any worker node, verify the kubelet invocation and configuration:
```bash
ps aux | grep kubelet
# e.g. /usr/bin/kubelet --kubeconfig=/etc/kubernetes/kubelet.conf \
#       --bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.conf \
#       --config=/var/lib/kubelet/config.yaml \
#       --cgroup-driver=systemd \
#       --network-plugin=cni
```

```bash
cat /var/lib/kubelet/config.yaml
# apiVersion: kubelet.config.k8s.io/v1beta1
# kind: KubeletConfiguration
# authentication:
#   anonymous:
#     enabled: false
#   x509:
#     clientCAFile: /path/to/ca.crt
# authorization:
#   mode: Webhook
# readOnlyPort: 0
# rotateCertificates: true
# staticPodPath: /etc/kubernetes/manifests
```

## 3. Kubelet API Endpoints

| Port   | Endpoint Type     | Access                     | Recommendation | 
|--------|-------------------|----------------------------|---------------|
| 10250  | Secure API        | TLS + AuthN/AuthZ required | Keep enabled and locked down |
| 10255  | Read-only metrics | Unauthenticated, HTTP only | Disable in production |

- Anyone with network access to port 10255 can scrape metrics:
```bash
curl -s http://localhost:10255/metrics | head -n 5
# HELP process_cpu_seconds_total Total user and system CPU time spent in seconds.
# TYPE process_cpu_seconds_total counter
# process_cpu_seconds_total 0.01
```

## 4. Authentication Configuration
- By default, the kubelet permits anonymous requests (`system:anonymous`). Disable this to force clients to present credentials.

### Disable Anonymous Access
- Via flags in your systemd unit:
```ini
--anonymous-auth=false \
--client-ca-file=/path/to/ca.crt
```

- Or in /var/lib/kubelet/config.yaml:
```yml
authentication:
  anonymous:
    enabled: false
  x509:
    clientCAFile: /path/to/ca.crt
```

### Certificate-Based Client Auth
1. Generate a CA and sign a kubelet-serving certificate.
2. Distribute the CA bundle with --client-ca-file=/path/to/ca.crt.
3. Test with:
    ```yml
    curl -s --key kubelet-key.pem --cert kubelet-cert.pem \
    https://localhost:10250/pods
    ```
4. Ensure the API Server has credentials to call the kubelet:
    ```ini
    # /etc/systemd/system/kube-apiserver.service
    --kubelet-client-certificate=/path/to/kubelet-client.crt \
    --kubelet-client-key=/path/to/kubelet-client.key
    ```

## 5. Authorization Modes
- Out of the box, the kubelet uses `AlwaysAllow` (no authorization). 
- Switch to `Webhook` to delegate decisions to the API Server.
    ```ini
    # Flags
    --authorization-mode=Webhook
    ```
    ```yml
    # config.yaml
    authorization:
    mode: Webhook
    ```
- Each kubelet request is then validated via the API Server’s SubjectAccessReview endpoint.

## 6. Disabling the Read-Only Port
- To completely turn off port 10255:
    ```ini
    # Flags
    --read-only-port=0
    ```
    ```yml
    # config.yaml
    readOnlyPort: 0
    ```

> [!WARNING]
> Always set `readOnlyPort: 0` in production to prevent unauthenticated access to metrics.

## 7. Summary of Hardening Steps

| Security Aspect      | Recommended Setting             |
|----------------------|---------------------------------|
| Anonymous Auth	   | `--anonymous-auth=false`        | 
| TLS Client AuthN     | `clientCAFile: /path/to/ca.crt` | 
| Authorization	       | `--authorization-mode=Webhook`  | 
| Read-Only Port       | `readOnlyPort: 0`               | 
| Certificate Rotation | `rotateCertificates: true`      | 

### Example Final kubelet.service Snippet
```ini
ExecStart=/usr/local/bin/kubelet \
  --config=/var/lib/kubelet/config.yaml \
  --anonymous-auth=false \
  --client-ca-file=/path/to/ca.crt \
  --authorization-mode=Webhook \
  --read-only-port=0
```

### Example Final `config.yaml`
```yml
apiVersion: kubelet.config.k8s.io/v1beta1
kind: KubeletConfiguration
authentication:
  anonymous:
    enabled: false
  x509:
    clientCAFile: /path/to/ca.crt
authorization:
  mode: Webhook
readOnlyPort: 0
rotateCertificates: true
```
