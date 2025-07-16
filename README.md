# 🛠️ Metrics Server TLS Cert Fix for Minikube on WSL2

### Fix to a problem I faced while setting up Kubernetes on WSL2

If you want to monitor memory and CPU usage of your pods and nodes in Kubernetes, you need **Metrics Server** running.

You can install it using:

```bash
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/high-availability-1.21+.yaml


This gives us API access to run commands like:

bash
Copy
Edit
kubectl top nodes
kubectl top pods

🧨 The Issue on WSL2 + Minikube
Sometimes this setup doesn’t work on WSL2 when using Minikube.

Here's why:

Metrics Server connects securely (HTTPS) to each node’s kubelet to get CPU/memory usage.

Kubelets use a TLS certificate for encryption.

On Minikube (especially inside WSL2), that cert is self-signed and missing proper IP SANs (Subject Alternative Names).

So Metrics Server refuses to connect because it can’t verify the certificate.

You’ll see an error in logs like this:

vbnet
Copy
Edit
tls: failed to verify certificate: x509: cannot validate certificate for <IP> because it doesn't contain any IP SANs
To check the logs:

bash
Copy
Edit
kubectl logs -n kube-system deployment/metrics-server
✅ The Fix
We fix this by telling Metrics Server to skip cert verification.

Run:

bash
Copy
Edit
kubectl edit deployment metrics-server -n kube-system
In the file that opens, scroll down to:

yaml
Copy
Edit
spec:
  containers:
    - name: metrics-server
      args:
Then add this line to the args: section:

yaml
Copy
Edit
--kubelet-insecure-tls
Your final args section might look like:

yaml
Copy
Edit
args:
  - --cert-dir=/tmp
  - --secure-port=10250
  - --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname
  - --kubelet-use-node-status-port
  - --metric-resolution=15s
  - --kubelet-insecure-tls
Save and exit the editor.

🔁 Restart and Test
After a few seconds, test it by running:

bash
Copy
Edit
kubectl top nodes
kubectl top pods
And check the API service status:

bash
Copy
Edit
kubectl get apiservice | grep metrics
You should see it say True now.

📌 Notes
This fix is only for local dev setups like WSL2 or Minikube.

In production, kubelets should use proper TLS certificates with valid SANs — this flag shouldn't be used there.



