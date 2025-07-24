# Client Security: `kubectl proxy` and `port-forward`

## 🛡️ 1. `kubectl` Security: Interacting with the Kubernetes API
`kubectl` is the primary CLI tool for interacting with the Kubernetes API.
- `kubectl` uses `kubeconfig` file stored at location `~/.kube/config` to authenticate and authorize users.
- If we call the API directly over HTTPS without credentials, we get a 403 error:
    ```bash
    $ curl https://<kube-api-server-ip>:6443 -k
    {
        "kind": "Status",
        "status": "Failure",
        "message": "forbidden: User \"system:anonymous\" cannot get path \"/\"",
        "code": 403
    }
    ```
- Supplying client certificates lets us authenticate:
    ```bash
    $ curl https://<kube-api-server-ip>:6443 -k \
    --key admin.key \
    --cert admin.crt \
    --cacert ca.crt
    ```
    This returns a list of available API paths:
    ```bash
    {
        "paths": [
            "/api/",
            "/api/v1/",
            "/apis/",
            "/healthz/",
            "/metrics/"
        ]
    }
    ```
    
### Comparison of Access Methods

| Method            | Command                   | Authentication  | Use Case | 
|-------------------|---------------------------|-----------------|----------|
| kubectl CLI       | `kubectl get nodes`       | kubeconfig      | Standard cluster management | 
| Direct API Curl   | `curl https://<api>:6443` | kubeconfig      | Scripting or debugging API interactions |
| kubectl proxy     | `kubectl proxy`           | kubeconfig      | Local HTTP proxy for API & services |

## 2. Using `kubectl proxy`
- The kubectl proxy command starts a local HTTP server (default port `8001`) that forwards requests to the API server using your kubeconfig credentials:
    ```bash
    kubectl proxy
    # Starting to serve on 127.0.0.1:8001
    ```
- Now we can access the API via http://localhost:8001:
    ```bash
    $ curl http://localhost:8001
    {
        "paths": [
            "/api",
            "/api/v1",
            "/apis",
            "/healthz",
            "/metrics",
            "/openapi/v2",
            "/swagger-2.0.0.json"
        ]
    }
    ```

> [!NOTE]
> By default, kubectl proxy listens only on the loopback interface (127.0.0.1) for security.

## 3. Accessing In-Cluster Services via Proxy
- We can also reach Services of type ClusterIP inside the cluster through the proxy. 
- For example, to access an NGINX Service in the default namespace:
    ```bash
    $ curl http://localhost:8001/api/v1/namespaces/default/services/nginx/proxy/

    <!DOCTYPE html>
    <html>
    <head><title>Welcome to nginx!</title></head>
    <body>
    <h1>Welcome to nginx!</h1>
    <p>…</p>
    </body>
    </html>
    ```
- With kubectl proxy, the in-cluster Service appears as if it’s running locally.

## 4. Port Forwarding with `kubectl port-forward`
- An alternative to proxying is **port forwarding**, which maps a local port directly to a Pod or Service port:
    ```bash
    kubectl port-forward service/nginx 8080:80
    ```
    - Local endpoint: `http://localhost:8080`
    - Cluster endpoint: Service `nginx` port `80`
- Now, visiting `http://localhost:8080` sends traffic through the API server to the `nginx` Service.
