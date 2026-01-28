# DevOps Infrastructure Cleanup Script

This repository contains a **bash script** to perform a full cleanup of DevOps tools, container runtimes, Kubernetes components, and related application data from a server.  
It is intended for environments where you want to remove all traces of DevOps infrastructure safely.

---

## Script: `devops-cleanup.sh`

```bash
#!/bin/bash
# DevOps Infrastructure Cleanup Script
# Usage: sudo bash devops-cleanup.sh

echo "=== STARTING DEVOPS INFRASTRUCTURE CLEANUP ==="
echo "Timestamp: $(date)"
echo "Hostname: $(hostname)"
echo ""

# 1. STOP ALL RUNNING CONTAINERS
echo "[1/9] Stopping all containers..."
sudo docker stop $(sudo docker ps -q) 2>/dev/null || echo "No Docker containers running"
sudo docker-compose down 2>/dev/null || echo "No Docker Compose projects"
sudo podman stop -a 2>/dev/null || echo "No Podman containers"
sudo k3s-killall.sh 2>/dev/null || echo "K3s not found"

# 2. STOP ALL DEVOPS SERVICES
echo "[2/9] Stopping DevOps services..."
sudo systemctl stop k3s-agent k3s docker containerd jenkins argocd-server argocd-repo-server \
    argocd-applicationset-controller rancher-server harbor nexus sonarqube gitlab-runner \
    vault consul nomad 2>/dev/null || true

sudo systemctl disable k3s-agent k3s docker containerd jenkins argocd-server argocd-repo-server \
    argocd-applicationset-controller rancher-server harbor nexus sonarqube gitlab-runner \
    vault consul nomad 2>/dev/null || true

# 3. REMOVE CONTAINER RUNTIMES
echo "[3/9] Removing container runtimes..."
sudo apt-get remove -y --purge docker.io docker-doc docker-compose docker-compose-v2 \
    podman-docker containerd runc docker-ce docker-ce-cli containerd.io \
    docker-buildx-plugin docker-compose-plugin 2>/dev/null || true

# 4. REMOVE KUBERNETES TOOLS
echo "[4/9] Removing Kubernetes tools..."
sudo rm -rf /usr/local/bin/{k3s,k3s-agent,k3s-killall,k3s-uninstall,kubectl,helm,k9s,kustomize,ctr,crictl}
sudo rm -rf /usr/bin/{kubectl,helm,k9s}
sudo rm -rf /opt/cni /opt/containerd

# 5. REMOVE DEVOPS APPLICATION DATA
echo "[5/9] Cleaning application data..."
sudo rm -rf /var/lib/{docker,containerd,rancher,kubelet,cri-dockerd}
sudo rm -rf /opt/{jenkins,argocd,harbor,rancher,nexus,sonarqube,gitlab}
sudo rm -rf ~/.{kube,docker,helm,minikube,argo}
sudo rm -rf /etc/{docker,containerd,rancher,kubernetes,cni}

# 6. REMOVE CONFIGURATION FILES
echo "[6/9] Removing config files..."
sudo rm -rf /etc/systemd/system/{k3s*,docker*,containerd*,jenkins*,argocd*,rancher*,harbor*}
sudo rm -rf /etc/apt/sources.list.d/{docker,kubernetes,rancher,jenkins,argoproj}.list
sudo rm -rf /usr/share/keyrings/{docker,kubernetes,rancher}*

# 7. CLEAN UP USERS & GROUPS
echo "[7/9] Cleaning users/groups..."
sudo deluser docker 2>/dev/null || true
sudo delgroup docker 2>/dev/null || true
sudo deluser jenkins 2>/dev/null || true
sudo deluser argocd 2>/dev/null || true

# 8. CLEAN PACKAGE MANAGEMENT
echo "[8/9] Cleaning package management..."
sudo apt-get autoremove -y
sudo apt-get autoclean -y
sudo apt-get update

# 9. RELOAD SYSTEMD & CLEAN TEMP
echo "[9/9] Final system cleanup..."
sudo systemctl daemon-reload
sudo systemctl reset-failed
sudo find /tmp -type f -atime +1 -delete 2>/dev/null || true
sudo find /var/tmp -type f -atime +1 -delete 2>/dev/null || true

echo ""
echo "=== DEVOPS CLEANUP COMPLETE ==="
echo "All container runtimes, orchestration tools, and DevOps applications removed."
