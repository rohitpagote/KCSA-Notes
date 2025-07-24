# Securing Etcd

- Etcd is the key-value store used by Kubernetes to store all its cluster data. 
- Because it contains the entire cluster state—including secrets—securing etcd is critically important.

## 🧠 What is etcd?
- A distributed, consistent key-value store.
- Stores cluster configuration, secrets, workload definitions, etc.
- Direct access to `etcd` = full control over the cluster.

## ⚠️ Risks if etcd is Unsecured
- Attackers can:
  - Extract secrets, tokens, and config data.
  - Modify cluster state.
  - Escalate privileges or take down the cluster.

## 🛡️ How to Secure etcd

### 1. Encrypting Data at Rest
- By default, etcd writes plaintext data to disk. 
- To safeguard sensitive objects - such as Secrets - enable Kubernetes built-in `EncryptionConfiguration`.

#### Step 1. Create an EncryptionConfiguration
- Save the following manifest as `encryption-config.yaml`:
    ```yml
    kind: EncryptionConfiguration
    apiVersion: apiserver.config.k8s.io/v1
    resources:
    - resources:
        - secrets
        providers:
        - aescbc:
            keys:
                - name: key1
                secret: <base64-encoded-encryption-key>
        - identity: {}
    ```

| Field     | Description             |
|-----------|---------------------------------|
| resources | List of resource types to encrypt (e.g., secrets, configmaps). | 
| providers | Ordered providers (mentioned below) | 

- `aescbc` uses AES-CBC. Replace `<base64-encoded-encryption-key>` with a 32-byte Base64 key.
- `identity` leaves data unencrypted as a fallback.

> [!NOTE]
> ## Generate a Strong Key
> Run the following command to create a 32-byte random key:
>   ```bash
>   openssl rand -base64 32
>   ```
> Copy the output into your encryption-config.yaml.

#### Step 2. Update the Etcd Static Pod
- Modify `/etc/kubernetes/manifests/etcd.yaml` to include the provider config:
    ```yml
    containers:
    - name: etcd
        image: k8s.gcr.io/etcd:3.4.13-0
        command:
        - etcd
        # ... other flags ...
        - --encryption-provider-config=/etc/kubernetes/encryption-config.yaml
    ```

### 2. Encrypting Data in Transit
- Protect etcd client-to-server and peer-to-peer communication with TLS certificates.

#### Step 1. Provision Certificates
- CA certificate (`ca.crt`)
- Server cert/key (`etcd-server.crt`, `etcd-server.key`)
- Peer cert/key (`etcd-peer.crt`, `etcd-peer.key`)
- Client cert/key (`etcd-client.crt`, `etcd-client.key`)

#### Step 2. Configure TLS Flags
- Extend etcd manifest:
    ```yml
    containers:
    - name: etcd
        image: k8s.gcr.io/etcd:3.4.13-0
        command:
        - etcd
        # Server TLS
        - --cert-file=/etc/etcd/tls/etcd-server.crt
        - --key-file=/etc/etcd/tls/etcd-server.key
        - --client-cert-auth
        - --trusted-ca-file=/etc/etcd/tls/ca.crt
        # Peer TLS
        - --peer-cert-file=/etc/etcd/tls/etcd-peer.crt
        - --peer-key-file=/etc/etcd/tls/etcd-peer.key
        - --peer-client-cert-auth
        - --peer-trusted-ca-file=/etc/etcd/tls/ca.crt
        # Encryption at rest
        - --encryption-provider-config=/etc/kubernetes/encryption-config.yaml
    ```

| Flag     | Purpose             |
|-----------|---------------------------------|
| --cert-file | Path to server TLS certificate for client connections | 
| --key-file | Path to server TLS private key | 
| --client-cert-auth | Require client certificates for authentication | 
| --trusted-ca-file | CA certificate to verify clients | 
| --peer-cert-file | TLS certificate for peer communication | 
| --peer-key-file  | TLS private key for peer communication | 
| --peer-client-cert-auth | Require peer certificates for mutual TLS | 
| --peer-trusted-ca-file | CA certificate to verify peer certificates | 

### 3. Backup and Disaster Recovery
- Regular snapshots of etcd are essential for restoring cluster state in case of data loss or corruption.

#### Taking a Snapshot
    ```bash
    ETCDCTL_API=3 etcdctl snapshot save /backups/etcd-snapshot.db \
    --endpoints=https://127.0.0.1:2379 \
    --cacert=/etc/etcd/tls/ca.crt \
    --cert=/etc/etcd/tls/etcd-client.crt \
    --key=/etc/etcd/tls/etcd-client.key
    ```

| Option     | Purpose             |
|-----------|---------------------------------|
| ETCDCTL_API=3 | Use the v3 etcdctl API | 
| -snapshot save | Command to write snapshot to disk | 
| --endpoints | Comma-separated list of etcd server URLs | 
| --cacert, --cert, --key | TLS credentials for authenticating with etcd | 

---

# ✅ Summary
Securing etcd involves:
1. **Encryption at Rest**
    - Use `EncryptionConfiguration` to encrypt Secrets (and other resources) on disk.
2. **Encryption in Transit**
    - Enforce TLS for all client and peer connections.
3. **Regular Backups**
    - Automate `etcdctl snapshot` to maintain up-to-date backups.
