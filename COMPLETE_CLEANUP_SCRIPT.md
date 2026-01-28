
## **2. COMPLETE_CLEANUP_SCRIPT.md**

```markdown
# Complete DevOps Cleanup Script

## Overview
This is the main cleanup script that removes all DevOps tools, container runtimes, and associated data from a system.

## Script: `cleanup-devops.sh`

```bash
#!/bin/bash
# DevOps Infrastructure Cleanup Script
# Complete removal of Docker, Kubernetes, CI/CD tools and all associated data
# Usage: sudo bash cleanup-devops.sh

set -e  # Exit on error

echo "=========================================="
echo "   DEVOPS INFRASTRUCTURE CLEANUP"
echo "=========================================="
echo "Timestamp: $(date)"
echo "Hostname: $(hostname)"
echo "User: $(whoami)"
echo ""

# Log file setup
LOG_FILE="/var/log/devops-cleanup-$(date +%Y%m%d-%H%M%S).log"
exec > >(tee -i "$LOG_FILE")
exec 2>&1

echo "Log file: $LOG_FILE"
echo ""

# ============================================================================
# PHASE 1: PRE-CHECKS AND WARNINGS
# ============================================================================
echo "=== PHASE 1: PRE-CHECKS ==="

# Check if running as root
if [ "$EUID" -ne 0 ]; then 
    echo "❌ ERROR: Please run as root (use sudo)"
    exit 1
fi

# Warning message
echo "⚠️  WARNING: This script will permanently remove:"
echo "   - All Docker containers and images"
echo "   - All Kubernetes clusters (K3s/K8s)"
echo "   - All CI/CD tools (Jenkins, ArgoCD, etc.)"
echo "   - All container data and configurations"
echo ""
read -p "Type 'YES' to continue: " CONFIRM
if [ "$CONFIRM" != "YES" ]; then
    echo "❌ Cleanup cancelled"
    exit 0
fi

# Backup important directories
echo "📦 Creating backups..."
BACKUP_DIR="/backup/devops-cleanup-$(date +%Y%m%d)"
mkdir -p "$BACKUP_DIR"
cp -r /etc/docker "$BACKUP_DIR/" 2>/dev/null || true
cp -r /etc/kubernetes "$BACKUP_DIR/" 2>/dev/null || true
cp -r /etc/rancher "$BACKUP_DIR/" 2>/dev/null || true
echo "✅ Backups created in: $BACKUP_DIR"
echo ""

# ============================================================================
# PHASE 2: STOP ALL RUNNING SERVICES AND CONTAINERS
# ============================================================================
echo "=== PHASE 2: STOPPING SERVICES ==="

# Stop all container runtimes
echo "🛑 Stopping container runtimes..."
systemctl stop docker containerd podman 2>/dev/null || true
systemctl disable docker containerd podman 2>/dev/null || true

# Stop Kubernetes services
echo "🛑 Stopping Kubernetes services..."
systemctl stop k3s k3s-agent kubectl kubelet kube-proxy 2>/dev/null || true
systemctl disable k3s k3s-agent kubectl kubelet kube-proxy 2>/dev/null || true

# Stop CI/CD services
echo "🛑 Stopping CI/CD services..."
systemctl stop jenkins argocd-server argocd-repo-server \
    argocd-applicationset-controller gitlab-runner 2>/dev/null || true
systemctl disable jenkins argocd-server argocd-repo-server \
    argocd-applicationset-controller gitlab-runner 2>/dev/null || true

# Stop other DevOps tools
echo "🛑 Stopping other DevOps services..."
systemctl stop harbor nexus sonarqube rancher-server 2>/dev/null || true
systemctl disable harbor nexus sonarqube rancher-server 2>/dev/null || true

# Kill any remaining processes
echo "🛑 Killing remaining processes..."
pkill -f "docker" 2>/dev/null || true
pkill -f "containerd" 2>/dev/null || true
pkill -f "k3s" 2>/dev/null || true
pkill -f "kube" 2>/dev/null || true
pkill -f "argocd" 2>/dev/null || true

# Stop all running containers (brute force)
echo "🛑 Stopping all containers..."
docker stop $(docker ps -q) 2>/dev/null || true
docker-compose down 2>/dev/null || true
podman stop -a 2>/dev/null || true

# K3s specific kill
if [ -f "/usr/local/bin/k3s-killall.sh" ]; then
    echo "🛑 Running K3s killall..."
    /usr/local/bin/k3s-killall.sh
fi

echo "✅ All services stopped"
echo ""

# ============================================================================
# PHASE 3: REMOVE CONTAINER RUNTIMES
# ============================================================================
echo "=== PHASE 3: REMOVING CONTAINER RUNTIMES ==="

# Remove Docker packages
echo "🗑️  Removing Docker packages..."
apt-get remove -y --purge \
    docker.io \
    docker-doc \
    docker-compose \
    docker-compose-v2 \
    docker-ce \
    docker-ce-cli \
    containerd.io \
    containerd \
    runc \
    podman \
    podman-docker \
    docker-buildx-plugin \
    docker-compose-plugin 2>/dev/null || true

# Remove Snap Docker
echo "🗑️  Removing Snap Docker..."
snap remove docker --purge 2>/dev/null || true

# Clean up Docker directories
echo "🗑️  Cleaning Docker directories..."
rm -rf /var/lib/docker
rm -rf /var/lib/containerd
rm -rf /var/lib/containers
rm -rf /var/lib/podman

# Clean up configuration
echo "🗑️  Removing Docker configuration..."
rm -rf /etc/docker
rm -rf /etc/containerd
rm -rf /etc/sysconfig/docker
rm -rf /etc/default/docker

# Remove Docker users and groups
echo "🗑️  Removing Docker users..."
deluser docker 2>/dev/null || true
delgroup docker 2>/dev/null || true

echo "✅ Container runtimes removed"
echo ""

# ============================================================================
# PHASE 4: REMOVE KUBERNETES COMPONENTS
# ============================================================================
echo "=== PHASE 4: REMOVING KUBERNETES ==="

# Remove K3s
echo "🗑️  Removing K3s..."
rm -rf /usr/local/bin/k3s
rm -rf /usr/local/bin/k3s-agent
rm -rf /usr/local/bin/k3s-killall.sh
rm -rf /usr/local/bin/k3s-uninstall.sh

# Remove Kubernetes binaries
echo "🗑️  Removing Kubernetes binaries..."
rm -rf /usr/local/bin/kubectl
rm -rf /usr/local/bin/helm
rm -rf /usr/local/bin/k9s
rm -rf /usr/local/bin/kustomize
rm -rf /usr/local/bin/ctr
rm -rf /usr/local/bin/crictl
rm -rf /usr/bin/kubectl
rm -rf /usr/bin/helm

# Remove Kubernetes data
echo "🗑️  Removing Kubernetes data..."
rm -rf /var/lib/rancher
rm -rf /var/lib/kubelet
rm -rf /var/lib/kubernetes
rm -rf /var/lib/cni
rm -rf /opt/cni
rm -rf /opt/containerd

# Remove configurations
echo "🗑️  Removing Kubernetes configurations..."
rm -rf /etc/rancher
rm -rf /etc/kubernetes
rm -rf /etc/cni
rm -rf ~/.kube
rm -rf ~/.helm
rm -rf ~/.minikube

# Remove systemd services
echo "🗑️  Removing Kubernetes systemd services..."
rm -rf /etc/systemd/system/k3s*
rm -rf /etc/systemd/system/kube*
rm -rf /lib/systemd/system/k3s*
rm -rf /lib/systemd/system/kube*

echo "✅ Kubernetes components removed"
echo ""

# ============================================================================
# PHASE 5: REMOVE CI/CD TOOLS
# ============================================================================
echo "=== PHASE 5: REMOVING CI/CD TOOLS ==="

# Remove Jenkins
echo "🗑️  Removing Jenkins..."
apt-get remove -y --purge jenkins 2>/dev/null || true
rm -rf /var/lib/jenkins
rm -rf /etc/jenkins
rm -rf /opt/jenkins
rm -rf /usr/share/jenkins
deluser jenkins 2>/dev/null || true

# Remove ArgoCD
echo "🗑️  Removing ArgoCD..."
rm -rf /usr/local/bin/argocd*
rm -rf /etc/argocd
rm -rf /opt/argocd
rm -rf ~/.argocd
rm -rf /etc/systemd/system/argocd*
deluser argocd 2>/dev/null || true

# Remove GitLab Runner
echo "🗑️  Removing GitLab Runner..."
apt-get remove -y --purge gitlab-runner 2>/dev/null || true
rm -rf /etc/gitlab-runner
rm -rf /var/lib/gitlab-runner

# Remove Harbor
echo "🗑️  Removing Harbor..."
rm -rf /opt/harbor
rm -rf /etc/harbor
rm -rf /data/harbor

# Remove Nexus
echo "🗑️  Removing Nexus..."
rm -rf /opt/sonatype/nexus
rm -rf /var/nexus

# Remove SonarQube
echo "🗑️  Removing SonarQube..."
rm -rf /opt/sonarqube
rm -rf /var/sonarqube

# Remove Rancher
echo "🗑️  Removing Rancher..."
rm -rf /opt/rancher
rm -rf /var/lib/rancher

echo "✅ CI/CD tools removed"
echo ""

# ============================================================================
# PHASE 6: CLEANUP MISCELLANEOUS
# ============================================================================
echo "=== PHASE 6: MISCELLANEOUS CLEANUP ==="

# Remove package repositories
echo "🗑️  Removing package repositories..."
rm -rf /etc/apt/sources.list.d/docker*.list
rm -rf /etc/apt/sources.list.d/kubernetes*.list
rm -rf /etc/apt/sources.list.d/rancher*.list
rm -rf /etc/apt/sources.list.d/jenkins*.list
rm -rf /etc/apt/sources.list.d/argoproj*.list

# Remove GPG keys
echo "🗑️  Removing GPG keys..."
rm -rf /usr/share/keyrings/docker*
rm -rf /usr/share/keyrings/kubernetes*
rm -rf /usr/share/keyrings/rancher*

# Clean up temp directories
echo "🗑️  Cleaning temporary directories..."
rm -rf /tmp/docker*
rm -rf /tmp/kube*
rm -rf /tmp/containerd*

# Clean up logs
echo "🗑️  Cleaning log files..."
rm -rf /var/log/docker*
rm -rf /var/log/containerd*
rm -rf /var/log/k3s*
rm -rf /var/log/kubernetes*
rm -rf /var/log/jenkins*
rm -rf /var/log/argocd*

echo "✅ Miscellaneous cleanup completed"
echo ""

# ============================================================================
# PHASE 7: SYSTEM CLEANUP
# ============================================================================
echo "=== PHASE 7: SYSTEM CLEANUP ==="

# Update package cache
echo "🔄 Updating package cache..."
apt-get update

# Autoremove orphaned packages
echo "🗑️  Removing orphaned packages..."
apt-get autoremove -y
apt-get autoclean -y

# Reload systemd
echo "🔄 Reloading systemd..."
systemctl daemon-reload
systemctl reset-failed

# Clean up temporary files
echo "🗑️  Cleaning temporary files..."
find /tmp -type f -atime +1 -delete 2>/dev/null || true
find /var/tmp -type f -atime +1 -delete 2>/dev/null || true

echo "✅ System cleanup completed"
echo ""

# ============================================================================
# PHASE 8: SECURITY HARDENING
# ============================================================================
echo "=== PHASE 8: SECURITY HARDENING ==="

# Enable firewall
echo "🔒 Enabling firewall..."
ufw --force enable 2>/dev/null || apt-get install -y ufw && ufw --force enable
ufw default deny incoming
ufw default allow outgoing
ufw allow ssh

# Block dangerous ports
echo "🔒 Blocking dangerous ports..."
ufw deny 10250   # Kubelet API
ufw deny 8472    # Flannel/WireGuard
ufw deny 6443    # Kubernetes API
ufw deny 30000:32767  # NodePort range
ufw reload

echo "✅ Security hardening completed"
echo ""

# ============================================================================
# COMPLETION
# ============================================================================
echo "=========================================="
echo "   CLEANUP COMPLETED SUCCESSFULLY"
echo "=========================================="
echo ""
echo "📊 SUMMARY:"
echo "✅ All container runtimes removed"
echo "✅ All Kubernetes components removed"
echo "✅ All CI/CD tools removed"
echo "✅ All configuration files cleaned"
echo "✅ Security hardening applied"
echo ""
echo "📋 NEXT STEPS:"
echo "1. Reboot the server: sudo reboot"
echo "2. Run verification script: sudo bash verify-cleanup.sh"
echo "3. Review log file: $LOG_FILE"
echo ""
echo "⚠️  IMPORTANT:"
echo "- Some manual cleanup might still be needed"
echo "- Check for any remaining processes with: ps aux | grep -E 'docker|kube|containerd'"
echo "- Verify firewall rules with: ufw status verbose"
echo ""
echo "🕒 Cleanup completed at: $(date)"
echo "=========================================="
