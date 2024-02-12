# Step 2: Install Ingress and KServe on MicroK8s

Now that MicroK8s is up and running, the next step is to install Ingress and KServe. Follow these steps to get everything set up:

## Enable Ingress in MicroK8s

Ingress controllers are vital for managing access to your services from outside the Kubernetes cluster. Enable Ingress by executing the following command:

```bash
microk8s enable ingress
```

This step allows you to route external traffic to your services inside the MicroK8s cluster.

Enable Cert-Manager
If you haven't enabled Cert-Manager in the previous step, please do so. Cert-Manager automates the management and issuance of TLS certificates:

```bash
microk8s enable cert-manager
```

# Enable Helm 3

Helm is a package manager for Kubernetes, making it easier to deploy and manage applications. Helm 3 is the latest version and doesn't require Tiller. Enable it by running:
```bash
microk8s enable helm3
```

# Configure kubectl and helm Aliases

To make command execution easier, set up aliases for microk8s.kubectl and microk8s.helm3. These aliases allow you to use kubectl and helm commands as if you were working with a standard Kubernetes cluster:

* For kubectl:
```bash
alias kubectl='microk8s.kubectl'
```

* For helm:
```bash
alias helm='microk8s.helm3'
```

This step ensures that you can use Helm to install and manage Kubernetes applications.

Install KServe
With all prerequisites in place, you're now ready to install KServe. KServe provides a serverless framework to deploy and manage machine learning models easily. Use the following command to apply the KServe runtime configuration:

```bash
kubectl apply -f https://github.com/kserve/kserve/releases/download/v0.11.0/kserve-runtimes.yaml
```

This command downloads and applies the KServe runtime configuration from the official GitHub releases.

By following these steps, you've successfully installed Ingress and KServe on your MicroK8s cluster. You're now ready to deploy machine learning models and manage access to them efficiently.

