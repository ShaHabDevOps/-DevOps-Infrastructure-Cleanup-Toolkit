=# DevOps Cleanup Verification Script

# these are the actual commands i ran while i was cleaning 

This repository contains a **bash script** to verify the cleanup status of a DevOps environment. It checks for container runtimes, Kubernetes tools, CI/CD tools, running processes, directory cleanup, and non-standard listening ports.

---

## Script: `verify_cleanup.sh`

```bash
#!/bin/bash
# Verification Script - Run after cleanup

echo "=== DEVOPS CLEANUP VERIFICATION ==="
echo ""

# 1. Check container runtimes
echo "1. CONTAINER RUNTIMES:"
echo "   Docker: $(which docker 2>/dev/null || echo '❌ NOT FOUND')"
echo "   Podman: $(which podman 2>/dev/null || echo '❌ NOT FOUND')"
echo "   Containerd: $(which containerd 2>/dev/null || echo '❌ NOT FOUND')"
echo ""

# 2. Check Kubernetes tools
echo "2. KUBERNETES TOOLS:"
echo "   kubectl: $(which kubectl 2>/dev/null || echo '❌ NOT FOUND')"
echo "   k3s: $(which k3s 2>/dev/null || echo '❌ NOT FOUND')"
echo "   helm: $(which helm 2>/dev/null || echo '❌ NOT FOUND')"
echo "   k9s: $(which k9s 2>/dev/null || echo '❌ NOT FOUND')"
echo ""

# 3. Check CI/CD tools
echo "3. CI/CD TOOLS:"
echo "   Jenkins: $(systemctl is-active jenkins 2>/dev/null || echo '❌ INACTIVE')"
echo "   ArgoCD: $(which argocd 2>/dev/null || echo '❌ NOT FOUND')"
echo "   GitLab Runner: $(systemctl is-active gitlab-runner 2>/dev/null || echo '❌ INACTIVE')"
echo ""

# 4. Check running processes
echo "4. RUNNING PROCESSES:"
echo "   Docker processes: $(ps aux | grep -E 'docker|containerd' | grep -v grep | wc -l)"
echo "   K8s processes: $(ps aux | grep -E 'kube|k3s' | grep -v grep | wc -l)"
echo "   ArgoCD processes: $(ps aux | grep 'argocd' | grep -v grep | wc -l)"
echo ""

# 5. Check directory cleanup
echo "5. DIRECTORY CLEANUP:"
directories="/var/lib/docker /var/lib/containerd /var/lib/rancher /opt/jenkins /opt/argocd"
for dir in $directories; do
    if [ -d "$dir" ]; then
        echo "   $dir: ⚠️  STILL EXISTS"
    else
        echo "   $dir: ✅ REMOVED"
    fi
done
echo ""

# 6. Check listening ports
echo "6. LISTENING PORTS (Non-standard):"
sudo ss -tulpn | grep -vE "(22|53|68|323|127.0.0|::1)" | awk '{print "   "$0}' || echo "   ✅ Only standard ports listening"
echo ""

echo "=== VERIFICATION COMPLETE ==="
