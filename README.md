# NKP v2.16.1 + Harbor Private Registry Deployment

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
![Platform](https://img.shields.io/badge/Platform-Nutanix%20AHV-orange)
![OS](https://img.shields.io/badge/OS-RHEL%2FRocky%209-red)
![NKP Version](https://img.shields.io/badge/NKP-v2.16.1-brightgreen)
![Harbor Version](https://img.shields.io/badge/Harbor-v2.13.5-blue)

A complete step-by-step deployment guide for deploying Nutanix Kubernetes Platform (NKP) v2.16.1 with Harbor Private Registry in an air-gapped environment on Nutanix AHV hypervisor.

---

## Table of Contents

- [Overview](#overview)
- [Architecture](#architecture)
- [Deployment Workflow](#deployment-workflow)
- [Prerequisites](#prerequisites)
- [Installation Steps](#installation-steps)
- [Configuration](#configuration)
- [Troubleshooting](#troubleshooting)
- [Best Practices](#best-practices)
- [Repository Structure](#repository-structure)
- [Contributing](#contributing)
- [License](#license)

---

## Overview

This deployment guide covers a complete, production-ready setup for:

- **NKP v2.16.1** — Nutanix Kubernetes Platform
- **Harbor v2.13.5** — Private container registry
- **CAPI Components** — Cluster API for Kubernetes management
- **Kommander** — Distributed Kubernetes operations platform
- **Air-Gapped Deployment** — No external internet access required
- **RHEL/Rocky Linux 9** — On Nutanix AHV hypervisor

### Key Features

✅ Air-gapped deployment with pre-bundled images  
✅ SSL/TLS certificates with custom CA  
✅ Bootstrap cluster creation and management  
✅ Node scaling and cluster operations  
✅ Complete offline image bundle deployment  
✅ Nutanix-native hypervisor integration  

---

## Architecture

The deployment consists of three main layers:

1. **Bastion VM** — Central deployment point (8 vCPU, 16 GB RAM, 300+ GB disk)
2. **Harbor Private Registry** — Air-gapped image storage with SSL/TLS
3. **NKP Management Cluster** — Kubernetes control plane with CAPI components

### Component Relationships

```
┌─────────────────────────────────────────────────────────────┐
│                    Nutanix AHV Cluster                      │
│                                                             │
│  ┌──────────────────┐         ┌──────────────────────────┐ │
│  │  Bastion VM      │         │  NKP Management Cluster  │ │
│  │  (Rocky Linux 9) │────────▶│  (Bootstrap + Mgmt)      │ │
│  │                  │         │                          │ │
│  │ • Docker Host    │         │ • Kubernetes            │ │
│  │ • NKP CLI        │         │ • CAPI Components       │ │
│  │ • kubectl        │         │ • Kommander             │ │
│  └──────────────────┘         └──────────────────────────┘ │
│         │                             ▲                     │
│         │ Pushes images to            │ Pulls from          │
│         ▼                             │                     │
│  ┌──────────────────────────────────────────────────────┐  │
│  │         Harbor Private Registry                       │  │
│  │  (10.48.107.198:443)                                 │  │
│  │                                                       │  │
│  │ • NKP Image Bundle (kommander-image-bundle)          │  │
│  │ • Bootstrap Images                                   │  │
│  │ • CAPI & System Images                               │  │
│  │ • SSL/TLS Certificates                               │  │
│  └──────────────────────────────────────────────────────┘  │
│         ▲                                                   │
│         │ Mounts                                            │
│         │ /data partition (300+ GB)                         │
│         │                                                   │
└─────────────────────────────────────────────────────────────┘
```

---

## Deployment Workflow

The deployment follows a structured sequence from infrastructure setup through cluster operations:

```
START
  │
  ├─► 1. Register OS & Install Dependencies
  │        └─ dnf packages, Docker, containerd
  │
  ├─► 2. Prepare Directory Structure
  │        └─ /nkp/bundles, /cert, /data
  │
  ├─► 3. Download & Extract Air-Gapped Bundles
  │        ├─ NKP v2.16.1 bundle
  │        └─ Harbor offline installer
  │
  ├─► 4. Load Docker Images
  │        ├─ konvoy-bootstrap-image
  │        └─ nkp-image-builder-image
  │
  ├─► 5. Install NKP CLI & kubectl
  │        └─ /usr/local/sbin/
  │
  ├─► 6. Create Bootstrap Cluster
  │        ├─ nkp create bootstrap
  │        └─ Verify: docker ps, kubectl get nodes
  │
  ├─► 7. Configure Storage for Harbor
  │        ├─ mkfs.ext4 /dev/sdb
  │        ├─ mount /dev/sdb /data
  │        └─ Persist in /etc/fstab
  │
  ├─► 8. Configure SSL/TLS Certificates
  │        ├─ Generate Root CA
  │        ├─ Generate Harbor CSR & certificate
  │        └─ Trust certificate in system
  │
  ├─► 9. Configure & Install Harbor
  │        ├─ Edit harbor.yml
  │        ├─ ./prepare
  │        └─ ./install.sh
  │
  ├─► 10. Verify Harbor Deployment
  │         └─ docker ps (3 containers: registry, proxy, core)
  │
  ├─► 11. Push NKP Image Bundle to Harbor
  │         └─ nkp push bundle → 10.48.107.198/nkp
  │
  ├─► 12. Generate & Configure SSH Keys
  │         └─ For node access & CAPI operations
  │
  ├─► 13. Deploy NKP Management Cluster
  │         └─ nkp create cluster
  │
  ├─► 14. Deploy CAPI Components
  │         └─ Cluster API for cluster lifecycle
  │
  ├─► 15. Configure Node Scaling
  │         └─ CAPI machine pool configuration
  │
  └─► 16. Upgrade Kommander (Optional)
         └─ Distributed platform operations
  │
  END
```

---

## Prerequisites

### Hardware Requirements

#### Bastion VM

| Resource | Minimum | Recommended |
|----------|---------|-------------|
| vCPU | 4 | 8 |
| RAM | 8 GB | 16 GB |
| Disk | 200 GB | 300+ GB |
| OS | RHEL 9 / Rocky 9 | Rocky Linux 9 |

#### Storage

- **Bastion root disk**: 300+ GB (NKP bundles + extracted images)
- **Harbor data volume**: 500 GB - 2 TB (image storage, scalable)
- **NKP cluster nodes**: 100 GB+ per node (OS + container runtime)

#### Network

- **Network access** between Bastion VM and NKP nodes
- **VLAN/network isolation** for air-gapped environment
- **DNS resolution** (internal) for registry hostname (e.g., `harbor.local` or IP-based)
- **HTTPS port 443** open for Harbor registry access

### Software Requirements

- Rocky Linux 9 or RHEL 9
- Docker 20.10+ with Docker Compose
- `wget` or `curl` for downloading bundles
- OpenSSL for certificate generation
- `subscription-manager` (RHEL only)

### Network Prerequisites (Air-Gapped)

- **Pre-downloaded bundles**: NKP v2.16.1 air-gapped bundle + Harbor offline installer
- **No outbound internet** required after initial downloads
- **Nitanix AHV cluster** with SSH access to Bastion VM

---

## Installation Steps

### Step 1: Register OS and Install Required Packages

```
# RHEL only: Register with subscription manager
subscription-manager register

# Install wget
yum install wget -y

# Add Docker repository and install
sudo dnf -y install dnf-plugins-core
sudo dnf config-manager --add-repo https://download.docker.com/linux/rhel/docker-ce.repo
sudo dnf install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y

# Enable and start Docker
systemctl enable --now docker
systemctl restart docker
systemctl status docker
```

### Step 2: Prepare Directory Structure

```
# Create NKP directories
mkdir -p /nkp/bundles
mkdir -p /cert
mkdir -p /data

cd /nkp/bundles
```

### Step 3: Download NKP Air-Gapped Bundle

```
# Download bundle (replace <NKP_BUNDLE_URL> with actual URL)
wget "<NKP_BUNDLE_URL>/nkp-air-gapped-bundle_v2.16.1_linux_amd64.tar.gz"

# Extract bundle
tar -xvzf nkp-air-gapped-bundle_v2.16.1_linux_amd64.tar.gz

# Navigate to bundle
cd nkp-v2.16.1
```

### Step 4: Load NKP Docker Images

```
# Load bootstrap image
docker load -i konvoy-bootstrap-image-v2.16.1.tar

# Load image-builder image
docker load -i nkp-image-builder-image-v2.16.1.tar

# Verify loaded images
docker images | grep -E 'konvoy|nkp'
```

### Step 5: Install NKP CLI and kubectl

```
cd /nkp/bundles/nkp-v2.16.1/cli/

# Install NKP binary
cp nkp /usr/local/sbin/
chmod +x /usr/local/sbin/nkp

# Install kubectl
cp kubectl /usr/local/sbin/
chmod +x /usr/local/sbin/kubectl

# Verify installation
which nkp
which kubectl
nkp version
kubectl version --client
```

### Step 6: Create NKP Bootstrap Cluster

```
cd /nkp/bundles/nkp-v2.16.1

# Create bootstrap cluster (this creates a local Kind cluster)
nkp create bootstrap

# Verify bootstrap cluster
docker ps -a
kubectl get nodes
kubectl get po -A
```

### Step 7: Configure Storage for Harbor

```
# Check available disks
lsblk

# Format additional disk (assuming /dev/sdb)
sudo mkfs.ext4 /dev/sdb

# Create mount point
sudo mkdir -p /data

# Mount disk
sudo mount /dev/sdb /data

# Verify
df -h

# Persist mount in /etc/fstab
sudo bash -c 'echo "/dev/sdb   /data   ext4   defaults   0 0" >> /etc/fstab'

# Reload
sudo systemctl daemon-reload
sudo mount -a
```

### Step 8: Generate SSL Certificates for Harbor

```
# Create certificate directory
mkdir -p /cert && cd /cert

# Generate Root CA private key
openssl genrsa -out rootCA.key 2048

# Generate Root CA certificate
openssl req -x509 -new -nodes \
  -key rootCA.key \
  -sha256 \
  -days 1024 \
  -out rootCA.pem \
  -subj "/C=IN/ST=Haryana/L=Gurgaon/O=NKP/OU=IT/CN=harbor-ca"

# Generate Harbor server private key
openssl genrsa -out ssl.key 2048

# Generate Harbor CSR
openssl req -new -key ssl.key -out ssl.csr \
  -subj "/C=IN/ST=Haryana/L=Gurgaon/O=NKP/OU=IT/CN=harbor.local"

# Create openssl.cnf for certificate extensions
cat > openssl.cnf << 'EOF'
[ req ]
default_bits       = 2048
prompt             = no
default_md         = sha256
req_extensions     = req_ext
distinguished_name = dn

[ dn ]
C=IN
ST=Haryana
L=Gurgaon
O=NKP
OU=IT
CN=harbor.local

[ req_ext ]
subjectAltName = @alt_names

[ alt_names ]
IP.1 = 10.10.x.x
DNS.1 = harbor.local

[ v3_req ]
subjectAltName = @alt_names
EOF

# Generate Harbor certificate signed by Root CA
openssl x509 -req \
  -in ssl.csr \
  -CA rootCA.pem \
  -CAkey rootCA.key \
  -CAcreateserial \
  -out ssl.crt \
  -days 825 \
  -sha256 \
  -extfile openssl.cnf \
  -extensions v3_req

# Trust certificate in system
sudo cp rootCA.pem /etc/pki/ca-trust/source/anchors/
sudo update-ca-trust extract

# Verify certificate
openssl x509 -in ssl.crt -text -noout
```

### Step 9: Download and Extract Harbor

```
cd /nkp/bundles/nkp-v2.16.1
mkdir -p harbor && cd harbor

# Download Harbor offline installer
wget https://github.com/goharbor/harbor/releases/download/v2.13.5/harbor-offline-installer-v2.13.5.tgz

# Extract
tar -xvzf harbor-offline-installer-v2.13.5.tgz

cd harbor
```

### Step 10: Configure Harbor

```
# Copy template
cp harbor.yml.tmpl harbor.yml

# Edit harbor.yml
vi harbor.yml
```

**Key configuration values to update:**

```yaml
# Harbor hostname or IP
hostname: 10.10.10.x.x

# HTTPS configuration
https:
  port: 443
  certificate: /cert/ssl.crt
  private_key: /cert/ssl.key

# Harbor admin password
harbor_admin_password: Harbor12345

# Data volume mount
data_volume: /data

# Logging (optional)
log:
  level: info
  local:
    rotate_count: 50
    rotate_size: G
    location: /var/log/harbor
```

### Step 11: Install and Start Harbor

```
# Prepare Harbor (generates docker-compose files)
./prepare

# Install Harbor
./install.sh

# Verify Harbor deployment
docker ps

# Check logs
docker-compose logs -f
```

Expected running containers:
- `harbor-registry` — Container image storage
- `harbor-core` — Web UI and API
- `harbor-proxy` — Reverse proxy

### Step 12: Push NKP Images to Harbor

```
cd /nkp/bundles/nkp-v2.16.1/container-images/

# Push image bundle to Harbor
nkp push bundle \
  --bundle /nkp/bundles/nkp-v2.16.1/container-images/kommander-image-bundle-v2.16.1.tar \
  --to-registry 10.10.10.x.x/nkp \
  --to-registry-username admin \
  --to-registry-password Harbor12345 \
  --to-registry-ca-cert-file /cert/rootCA.pem

# Verify images are available in Harbor
# Access Harbor UI: https://10.10.x.x
# Login: admin / Harbor12345
```

### Step 13: Generate SSH Keys (Optional but Recommended)

```
# Generate SSH key for cluster operations
ssh-keygen -t ed25519 -f ~/.ssh/nkp-cluster-key -N ""

# Add public key to known hosts
cat ~/.ssh/nkp-cluster-key.pub
```

---

## Configuration

### Harbor SSL/TLS Configuration

Harbor uses self-signed certificates by default. For production, consider:

1. **Certificate Rotation**: Every 825 days (2+ years)
2. **Certificate Validation**: Always verify against your Root CA
3. **Client Configuration**: Ensure client systems trust the Root CA:

```
# On client systems, trust the Harbor CA
sudo cp /cert/rootCA.pem /etc/pki/ca-trust/source/anchors/harbor-ca.pem
sudo update-ca-trust extract

# Verify Docker can connect to Harbor
docker login -u admin -p Harbor12345 10.x.x.x
```

### NKP Cluster Configuration

Default bootstrap cluster details:

- **Cluster Name**: `konvoy-bootstrap`
- **Container Runtime**: containerd
- **CNI**: Calico (default)
- **Kube Version**: v1.27+ (bundled)
- **CAPI Support**: Yes (for node scaling and cluster lifecycle)

### Registry Configuration for Nodes

All NKP nodes must trust the Harbor CA certificate:

```
# Distribute rootCA.pem to all cluster nodes
scp /cert/rootCA.pem node-ip:/etc/pki/ca-trust/source/anchors/
ssh node-ip "sudo update-ca-trust extract"
```

---

## Troubleshooting

### Docker Connection Issues

**Problem**: `docker: command not found`

**Solution**:
```
# Verify Docker installation
systemctl status docker
which docker

# Restart Docker if needed
sudo systemctl restart docker
```

### Harbor Startup Failures

**Problem**: Harbor containers fail to start

**Solution**:
```
# Check logs
docker-compose -f /path/to/harbor/docker-compose.yml logs

# Verify certificate paths
ls -la /cert/ssl.crt /cert/ssl.key

# Check port availability
netstat -tlnp | grep 443

# Restart Harbor
docker-compose -f /path/to/harbor/docker-compose.yml restart
```

### Image Push Failures

**Problem**: `docker push 10.x.1.x/nkp/image fails`

**Solution**:
```
# Verify Harbor is running
curl -k https://10.x.1.x/api/v2.0/projects

# Check Docker login
docker login 10.48.1x.x

# Verify certificate trust
curl -vk https://10.x.1.x/api/v2.0/projects

# Check DNS resolution (if using hostname)
nslookup harbor.local
```

### NKP Bootstrap Cluster Issues

**Problem**: `nkp create bootstrap` fails

**Solution**:
```
# Clean up previous bootstrap
docker ps -a | grep konvoy-bootstrap | awk '{print $1}' | xargs docker rm -f

# Check Docker images are loaded
docker images | grep konvoy

# Verify disk space
df -h /

# Try again with verbose output
nkp create bootstrap --verbose
```

### Certificate Trust Issues

**Problem**: `x509: certificate signed by unknown authority`

**Solution**:
```
# Verify Root CA is trusted
cat /etc/pki/ca-trust/source/anchors/rootCA.pem

# If missing, add and update trust
sudo cp /cert/rootCA.pem /etc/pki/ca-trust/source/anchors/
sudo update-ca-trust extract

# Verify Docker daemon trusts certificate
docker exec <harbor-container> curl -v https://localhost
```

---

## Best Practices

### Security

- **Passwords**: Change default Harbor admin password (`Harbor12345`) in production
- **RBAC**: Configure Harbor RBAC policies for image access control
- **Network**: Isolate Harbor on a dedicated network segment
- **Certificates**: Use production-grade certificates; rotate regularly
- **Audit**: Enable Harbor audit logging for compliance

### Operations

- **Backup**: Regularly backup `/data/database` and Harbor configuration
- **Monitoring**: Monitor Harbor storage usage and capacity
- **Updates**: Test Harbor and NKP updates in non-prod environments first
- **Documentation**: Maintain cluster topology and configuration runbooks

### Performance

- **Storage**: Use fast SSD storage for Harbor `/data` volume
- **Network**: Ensure high-bandwidth connectivity between Bastion and cluster nodes
- **CPU/Memory**: Monitor Bastion VM resource usage during image pushes
- **Image Cleanup**: Periodically clean up unused images with `nkp prune`

---

## Repository Structure

```
nkp-harbor-deployment/
├── README.md                          # This file
├── scripts/                           # Automation scripts
│   ├── 01-setup-bastion.sh          # Bastion VM setup
│   ├── 02-prepare-storage.sh         # Storage configuration
│   ├── 03-setup-certificates.sh      # SSL/TLS certificate generation
│   ├── 04-install-harbor.sh          # Harbor installation
│   └── 05-deploy-nkp.sh              # NKP bootstrap and cluster deployment
├── configs/                           # Configuration templates
│   ├── harbor.yml.template           # Harbor configuration template
│   ├── openssl.cnf                   # OpenSSL certificate config
│   └── docker-compose.override.yml   # Harbor Docker Compose overrides
├── docs/                              # Additional documentation
│   ├── ARCHITECTURE.md               # Detailed architecture guide
│   ├── TROUBLESHOOTING.md            # Troubleshooting guide
│   ├── SSL-CERTIFICATE-RENEWAL.md    # Certificate renewal procedures
│   └── CLUSTER-OPERATIONS.md         # Cluster scaling and operations
└── examples/                          # Example files
    ├── cluster-config-example.yaml   # NKP cluster configuration example
    └── capi-machine-pool.yaml        # CAPI machine pool definition
```

---

## Contributing

Contributions are welcome! Please follow these guidelines:

1. **Fork** this repository
2. **Create** a feature branch (`git checkout -b feature/improvement`)
3. **Make** your changes with clear commit messages
4. **Test** your changes in an air-gapped environment
5. **Submit** a Pull Request with detailed description



---

## Support & Community

For questions and support:

- **Issues**: Report bugs or request features via GitHub Issues
- **Discussions**: Join community discussions for best practices
- **Documentation**: Check [docs/](docs/) for additional guides
- **Nutanix Community**: Visit [Nutanix Developer Community](https://www.nutanixdev.com/)

---

## Changelog

### v1.0.0 (Current)

- Initial release with NKP v2.16.1 and Harbor v2.13.5
- Complete air-gapped deployment guide
- SSL/TLS certificate generation procedures
- Troubleshooting and best practices
- Repository structure and automation scripts

---

**Last Updated**: June 2026  
**Maintained By**: **Shivang Bhardwaj**
