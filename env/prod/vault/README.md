# HashiCorp Vault Deployment

This directory contains configurations for deploying HashiCorp Vault using either direct volume deployment or Helm chart.

## Overview

The deployment consists of:
- Vault server deployment (Helm or direct)
- Local storage configuration
- Service account and authentication setup
- Policy configurations
- Audit logging

## Prerequisites

1. Kubernetes cluster (v1.25+)
2. Helm v3 (for Helm deployment)
3. Local storage provisioner
4. kubectl access with cluster-admin privileges

## Deployment Options

### Option 1: Volume-Based Deployment

#### 1. Storage Setup
```bash
# On the designated node
sudo mkdir -p /mnt/vault
sudo chmod 777 /mnt/vault

# Create storage resources
kubectl apply -f storageClass.yaml
kubectl apply -f vault.yaml
```

#### 2. Deploy Vault
```bash
# Create namespace
kubectl create namespace vault

# Deploy Vault
kubectl apply -f vault.yaml
```

### Option 2: Helm Deployment (Recommended)

#### 1. Add Vault Helm Repository
```bash
helm repo add hashicorp https://helm.releases.hashicorp.com
helm repo update
```

#### 2. Deploy Using Helm
```bash
# Create namespace
kubectl create namespace vault

# Install Vault using our custom values
helm install vault hashicorp/vault \
  --namespace vault \
  --values values.yaml
```

Our `values.yaml` includes:
- Custom resource limits
- Local storage configuration
- Audit logging setup
- Service configurations

## Configuration

### Initialize Vault
```bash
# Get the pod name
export VAULT_POD=$(kubectl get pod -n vault -l app.kubernetes.io/name=vault -o jsonpath='{.items[0].metadata.name}')

# Initialize Vault
kubectl exec -n vault $VAULT_POD -- vault operator init
```

### Configure Policies
```bash
# Apply policies
kubectl cp my-app-policy.hcl vault/$VAULT_POD:/tmp/
kubectl exec -n vault $VAULT_POD -- vault policy write my-app /tmp/my-app-policy.hcl
```

## Storage Configuration

### Local Storage
- **PersistentVolume**:
  - Size: 10Gi
  - Path: `/mnt/vault`
  - Storage Class: local-storage
  - Access Mode: ReadWriteOnce
  - Node Affinity: Configured to specific node

### Volume Configuration (values.yaml)
```yaml
dataStorage:
  enabled: true
  storageClass: "local-storage"
  accessMode: ReadWriteOnce
  size: 10Gi
```

## Usage

### Accessing Vault

1. **Port Forward**:
```bash
kubectl port-forward -n vault svc/vault 8200:8200
```

2. **Environment Setup**:
```bash
export VAULT_ADDR='http://localhost:8200'
```

### Managing Secrets

1. **Write Secrets**:
```bash
vault kv put secret/my-app/config key=value
```

2. **Read Secrets**:
```bash
vault kv get secret/my-app/config
```

## Maintenance

### Backup
```bash
kubectl exec -n vault $VAULT_POD -- vault operator raft snapshot save /tmp/vault-backup.snap
kubectl cp vault/$VAULT_POD:/tmp/vault-backup.snap ./vault-backup.snap
```

### Upgrading

#### For Helm Deployment:
```bash
helm upgrade vault hashicorp/vault \
  --namespace vault \
  --values values.yaml
```

#### For Volume Deployment:
```bash
kubectl apply -f vault.yaml
```

### Audit Logs
```bash
kubectl logs -n vault $VAULT_POD
```

## Troubleshooting

### Common Issues

1. **Vault Status**:
```bash
kubectl exec -n vault $VAULT_POD -- vault status
```

2. **Storage Issues**:
```bash
kubectl describe pv vault-local-pv
kubectl describe pvc vault-pvc -n vault
```

3. **Pod Status**:
```bash
kubectl get pods -n vault
kubectl describe pod $VAULT_POD -n vault
```

4. **Helm Status** (for Helm deployment):
```bash
helm status vault -n vault
```

## Security Considerations

- Use TLS for production deployments
- Regularly rotate root tokens
- Enable audit logging
- Implement least privilege access
- Regular backup of Vault data
- Monitor for suspicious activities
- Use Kubernetes service accounts for authentication

## Support

For issues and support:
1. Check [HashiCorp Vault Documentation](https://www.vaultproject.io/docs)
2. Review Kubernetes events: `kubectl get events -n vault`
3. Check Vault logs: `kubectl logs -n vault $VAULT_POD`
4. Vault status: `kubectl exec -n vault $VAULT_POD -- vault status`
5. Helm documentation: [Vault Helm Chart](https://github.com/hashicorp/vault-helm)
