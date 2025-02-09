# MinIO Deployment

This directory contains Kubernetes manifests for deploying MinIO, a high-performance object storage system compatible with Amazon S3.

## Overview

The deployment consists of:
- MinIO server deployment
- Services for API and Console access
- Ingress configurations
- TLS certificates management
- Vault integration for secrets

## Components

### MinIO Server
- Image: `quay.io/minio/minio:latest`
- Resources:
  - Requests: 1024Mi RAM, 500m CPU
  - Limits: 2048Mi RAM, 1000m CPU
- Ports:
  - Console: 9001
  - API: 9000

### Services
1. **Console Service (minio-app-service)**
   - Type: ClusterIP
   - Port: 80 → 9001

2. **API Service (minio-app-service-api)**
   - Type: ClusterIP
   - Port: 80 → 9000

### Ingress
1. **Console Ingress**
   - Host: minio.smv.pp.ua
   - TLS enabled with cert-manager
   - Max body size: 1000m
   - WebSocket support enabled

2. **API Ingress**
   - Host: api.minio.smv.pp.ua
   - TLS enabled with cert-manager

### Vault Integration
- ServiceAccount: `minio-sa`
- Authentication: Kubernetes method
- Secret Path: `kv/minio`
- Refresh Interval: 30s

## TLS Certificates
Managed by cert-manager with Let's Encrypt:
- Console: minio.smv.pp.ua
- API: api.minio.smv.pp.ua
- Validity: 2160h (90 days)
- Auto-renewal: 1070h before expiry

## Usage

### Prerequisites
1. Kubernetes cluster
2. Helm v3
3. cert-manager installed
4. HashiCorp Vault configured
5. Ingress controller (nginx)

### Deployment
```bash
# Create namespace
kubectl create namespace minio

# Apply manifests
kubectl apply -f vaultSecrets.yaml
kubectl apply -f certificate.yaml
kubectl apply -f minio.yaml
```

### Access
- Console UI: https://minio.smv.pp.ua
- API Endpoint: https://api.minio.smv.pp.ua

## Security Considerations
- TLS encryption enabled for all endpoints
- Credentials managed through HashiCorp Vault
- Regular certificate rotation
- Ingress configured with secure headers
- Resource limits enforced

## Maintenance

### Certificate Renewal
Certificates are automatically renewed by cert-manager.

### Updating MinIO
```bash
kubectl -n minio set image deployment/minio-app minio=quay.io/minio/minio:DESIRED_VERSION
```

### Scaling
The deployment can be scaled horizontally if needed:
```bash
kubectl -n minio scale deployment minio-app --replicas=DESIRED_COUNT
```

## Troubleshooting

### Common Issues
1. Certificate Issues
```bash
kubectl -n minio get certificate
kubectl -n minio describe certificate minio-cert
```

2. Vault Integration
```bash
kubectl -n minio get vaultauth
kubectl -n minio describe vaultstaticsecret vault-kv-minio-v1
```

3. Pod Status
```bash
kubectl -n minio get pods
kubectl -n minio describe pod minio-app-XXXXX
```

## Support
For issues and support:
1. Check MinIO documentation: https://min.io/docs/minio/kubernetes/upstream/
2. Review Kubernetes events: `kubectl -n minio get events`
3. Check pod logs: `kubectl -n minio logs -f deployment/minio-app`
```
