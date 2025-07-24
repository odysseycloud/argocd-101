# argocd-101
argocd 101 walkthrough and demo

## Prerequisites

### Install Nix

Install Nix package manager:

@https://nixos.org/download/#nix-install-macos (macOS)
@https://nixos.org/download/#nix-install-windows (Windows WSL2)

```bash
sh <(curl --proto '=https' --tlsv1.2 -L https://nixos.org/nix/install)
```

## Setup Environment

Run the following command to set up the environment:

@https://search.nixos.org/packages - This is where you can find packages you want to install

```bash
nix-shell -p nix-info git docker kind kubernetes-helm kubectl --run $SHELL
```

## Start Docker Daemon

Since we're using Docker from nix, you need to start the daemon manually:

```bash
# Start Docker daemon in background
sudo dockerd &

# Wait a moment for daemon to start, then verify
sleep 3
docker info
```

**Alternative: Use systemd (if available in your WSL)**
```bash
# If systemd is available in your WSL
sudo systemctl start docker
sudo systemctl enable docker  # To start automatically
```

**For WSL users without systemd:**
- The `sudo dockerd &` method is recommended (had to explicitly specify path)
- You'll need to start the daemon each time you start a new WSL session
- Or you can add it to your shell profile (.bashrc, .zshrc) to start automatically

## Create Kubernetes Cluster

```bash
kind create cluster --name argo-101
```

**Troubleshooting kind issues:**
- Ensure Docker daemon is running (`docker info` should work)
- If cluster creation fails, try cleaning up first:
  ```bash
  kind delete cluster --name argo-101
  docker system prune -f
  ```
- Try with a specific node image:
  ```bash
  kind create cluster --name argo-101 --image kindest/node:v1.31.0
  ```
kind get kubeconfig --name argo-101
## Install ArgoCD

```bash
helm repo add argo https://argoproj.github.io/argo-helm
helm install argo-cd-101 argo/argo-cd --create-namespace --namespace cd
kubectl -n cd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d

kubectl port-forward service/argo-cd-101-argocd-server -n cd 8080:443

argocd login localhost:8080


argocd app create helm-guestbook --repo https://github.com/argoproj/argocd-example-apps.git --path helm-guestbook --dest-namespace default --dest-server https://kubernetes.default.svc --helm-set replicaCount=2

enabl auto sync
```
