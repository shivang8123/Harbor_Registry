# Quick Start Guide

For a complete guide, see [README.md](README.md). This is a condensed reference for experienced operators.

## Prerequisites Checklist

- [ ] Bastion VM: 8 vCPU, 16 GB RAM, 300+ GB disk (Rocky/RHEL 9)
- [ ] NKP v2.16.1 air-gapped bundle downloaded
- [ ] Harbor v2.13.5 offline installer downloaded
- [ ] Network access from Bastion to Nutanix AHV cluster
- [ ] Additional disk for Harbor storage (/dev/sdb, 500GB+)

## 5-Minute Setup

### 1. OS Setup (5 min)
```bash
# Register OS
subscription-manager register

# Install Docker
sudo dnf -y install dnf-plugins-core
sudo dnf config-manager --add-repo https://download.docker.com/linux/rhel/docker-ce.repo
sudo dnf install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y
systemctl enable --now docker
```

### 2. Prepare Directories (1 min)
```bash
mkdir -p /nkp/bundles /cert /data
cd /nkp/bundles
```

### 3. Extract Bundles (2 min)
```bash
tar -xzf nkp-air-gapped-bundle_v2.16.1_linux_amd64.tar.gz
tar -xzf harbor-offline-installer-v2.13.5.tgz -C nkp-v2.16.1/
```

### 4. Load Images (3 min)
```bash
cd nkp-v2.16.1
docker load -i konvoy-bootstrap-image-v2.16.1.tar
docker load -i nkp-image-builder-image-v2.16.1.tar
```

### 5. Install CLI Tools (1 min)
```bash
cp cli/nkp /usr/local/sbin/ && chmod +x /usr/local/sbin/nkp
cp cli/kubectl /usr/local/sbin/ && chmod +x /usr/local/sbin/kubectl
which nkp kubectl
```

## Full Deployment Timeline

| Phase | Duration | Tasks |
|-------|----------|-------|
| OS Setup | 5 min | Register, install Docker, enable service |
| Prep | 5 min | Create directories, extract bundles |
| Images | 10 min | Load Docker images, install CLI |
| Bootstrap | 15 min | Create bootstrap cluster, verify |
| Storage | 10 min | Format disk, mount, persist /etc/fstab |
| Certificates | 10 min | Generate Root CA, server cert, sign |
| Harbor | 20 min | Configure, install, verify containers |
| Image Push | 30 min | Push NKP bundle to Harbor registry |
| **Total** | **~2-3 hours** | Complete setup |

## Essential Commands

```bash
# Create bootstrap cluster
nkp create bootstrap

# Verify bootstrap
kubectl get nodes
docker ps -a

# Format and mount storage
sudo mkfs.ext4 /dev/sdb
sudo mount /dev/sdb /data
sudo bash -c 'echo "/dev/sdb /data ext4 defaults 0 0" >> /etc/fstab'

# Generate certificates
cd /cert
openssl genrsa -out rootCA.key 2048
openssl req -x509 -new -nodes -key rootCA.key -sha256 -days 1024 -out rootCA.pem
openssl genrsa -out ssl.key 2048
openssl req -new -key ssl.key -out ssl.csr

# Configure Harbor
cd /nkp/bundles/nkp-v2.16.1/harbor/harbor
cp harbor.yml.tmpl harbor.yml
vi harbor.yml  # Edit hostname, HTTPS certs, password, data_volume

# Install Harbor
./prepare
./install.sh
docker ps  # Verify: registry, core, proxy containers running

# Push images to Harbor
cd /nkp/bundles/nkp-v2.16.1/container-images/
nkp push bundle \
  --bundle kommander-image-bundle-v2.16.1.tar \
  --to-registry 10.48.107.198/nkp \
  --to-registry-username admin \
  --to-registry-password Harbor12345 \
  --to-registry-ca-cert-file /cert/rootCA.pem

# Deploy management cluster
nkp create cluster
```

## Harbor Quick Access

**URL:** `https://10.48.107.198`  
**Username:** `admin`  
**Password:** `Harbor12345` (change in production)

## Verification Checklist

```bash
# 1. Docker is running
systemctl status docker

# 2. Bootstrap cluster is healthy
kubectl get nodes
kubectl get po -A

# 3. Harbor containers are running
docker ps | grep harbor

# 4. Images are in Harbor
curl -k -u admin:Harbor12345 https://10.48.107.198/api/v2.0/projects

# 5. Storage is mounted
df -h | grep /data

# 6. Certificates are valid
openssl x509 -in /cert/ssl.crt -text -noout
```

## Common Issues

| Issue | Solution |
|-------|----------|
| Docker not found | `systemctl restart docker` |
| Harbor won't start | Check logs: `docker-compose logs -f` |
| Images won't push | Verify Harbor login: `docker login 10.48.107.198` |
| Bootstrap cluster fails | Clean up: `docker rm -f konvoy-bootstrap-*` then retry |
| Certificate errors | Trust Root CA: `sudo cp /cert/rootCA.pem /etc/pki/ca-trust/source/anchors/` then `sudo update-ca-trust extract` |

## Next Steps

1. Deploy NKP management cluster: `nkp create cluster`
2. Configure CAPI for node scaling
3. Deploy Kommander for multi-cluster management
4. Configure image garbage collection and retention policies

## Documentation

- [Full README](README.md) — Complete deployment guide
- [Architecture Guide](docs/ARCHITECTURE.md) — Detailed system design
- [Troubleshooting](docs/TROUBLESHOOTING.md) — Common problems and solutions
- [Certificate Renewal](docs/SSL-CERTIFICATE-RENEWAL.md) — SSL/TLS management

## Support

- Report issues: [GitHub Issues](../../issues)
- Questions: Check [Troubleshooting](docs/TROUBLESHOOTING.md)
- Community: [Nutanix Developer Community](https://www.nutanixdev.com/)

---

**Version:** 1.0.0 | **Updated:** June 2026
