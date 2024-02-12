# Step 1: Install MicroK8s on Ubuntu

Install MicroK8s on Ubuntu using Snap package manager.

## Install MicroK8s

Enter the following command in the terminal. This command installs MicroK8s and marks it as a classic snap, allowing it to function with the necessary permissions and access:

```bash
sudo snap install microk8s --classic
```

# Enable Essential Services
MicroK8s comes with several services that can be enabled as needed. For a development environment working with PDK, enabling DNS, storage, cert-manager, and GPU support is often necessary. Execute the following commands one by one to enable these services:

* Enable DNS:

```bash
sudo microk8s enable dns
```
* Enable Storage:
```bash
sudo microk8s enable storage
```
* Enable Cert-Manager:
```bash
sudo microk8s enable cert-manager
```
* Enable GPU (if you have a compatible GPU and need it for your development work):
```bash
sudo microk8s enable gpu
```
# Configure kubectl
kubectl is a command-line tool for interacting with Kubernetes clusters. MicroK8s has its own version of kubectl for convenience, but if you prefer to use the standalone version of kubectl or integrate MicroK8s with an existing Kubernetes workflow, you'll need to add the MicroK8s configuration to your ~/.kube/config file. Execute the following command to do so:

```bash
sudo microk8s kubectl config view --raw > ~/.kube/config
```
This command exports the current MicroK8s configuration into the default kubectl configuration file, allowing you to manage MicroK8s with kubectl.

Congratulations! You've successfully installed MicroK8s on Ubuntu and configured it for development with PDK. You're now ready to proceed with the next steps, such as installing PDK or deploying your applications.

