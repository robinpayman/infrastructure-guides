# Infrastructure Guides

Centralized documentation for infrastructure, deployment, and operational procedures across all projects.

## Overview

This repository contains comprehensive infrastructure guides for each project, covering:
- **Architecture & Deployment**: Where code runs, how it's deployed
- **Storage & Access**: Where secrets, credentials, and configurations are stored
- **Processes & Procedures**: How to deploy, monitor, and troubleshoot
- **Quick References**: Commands, URLs, environment variables

## Guides by Project

### 1. **Entrynor Website** (`entrynor-website`)
- **Repository**: https://github.com/robinpayman/entrynor-website
- **Live URL**: https://entrynor.no
- **Guide**: [entrynor-website-INFRASTRUCTURE_GUIDE.md](./entrynor-website-INFRASTRUCTURE_GUIDE.md)

**Stack**: Astro 7.1.3, Vercel, Resend (email), reCAPTCHA  
**Key Components**: Static site, bilingual (NO/EN), contact form with email delivery  
**Deployment**: Auto-deploys from GitHub `main` branch to Vercel

---

### 2. **Tripletex PDF Integration** (`tripletex-pdf-integration`)
- **Repository**: https://github.com/robinpayman/tripletex-pdf-integration
- **Deployment**: Azure VM with SFTP watcher service
- **Guide**: [tripletex-pdf-integration-INFRASTRUCTURE_GUIDE.md](./tripletex-pdf-integration-INFRASTRUCTURE_GUIDE.md)

**Stack**: Python, SFTP, Tripletex API, systemd service  
**Key Components**: PDF processing, invoice validation, duplicate detection, email alerts  
**Deployment**: 24/7 Linux service on Azure VM

---

### 3. **WooCommerce-Tripletex Integration** (`wc-tripletex-integration`)
- **Repository**: https://github.com/robinpayman/wc-tripletex-integration
- **Deployment**: WordPress plugin + Vercel API backend
- **Guide**: [wc-tripletex-integration-INFRASTRUCTURE_GUIDE.md](./wc-tripletex-integration-INFRASTRUCTURE_GUIDE.md)

**Stack**: PHP (WordPress), Vercel, MySQL, REST APIs  
**Key Components**: Order sync, product sync, stock sync, webhook integration  
**Deployment**: Auto-syncs when orders marked as "Processing"

---

### 4. **Klubben SFTP Service** (`klubben-sftp-service`)
- **Repository**: https://github.com/robinpayman/klubben-sftp-service
- **Deployment**: Azure VM with SFTP watcher service
- **Guide**: [klubben-sftp-service-INFRASTRUCTURE_GUIDE.md](./klubben-sftp-service-INFRASTRUCTURE_GUIDE.md)

**Stack**: Python, XML parsing, SFTP, Tripletex API  
**Key Components**: Klubben XML processing, customer mapping, duplicate detection  
**Deployment**: 24/7 Linux service on Azure VM (similar to PDF integration)

---

## Quick Access

| Project | Live URL | Status | Guide |
| --- | --- | --- | --- |
| **entrynor-website** | https://entrynor.no | ✅ Live | [View](./entrynor-website-INFRASTRUCTURE_GUIDE.md) |
| **tripletex-pdf-integration** | Azure VM | ✅ Deployed | [View](./tripletex-pdf-integration-INFRASTRUCTURE_GUIDE.md) |
| **wc-tripletex-integration** | Vercel API | ✅ Deployed | [View](./wc-tripletex-integration-INFRASTRUCTURE_GUIDE.md) |
| **klubben-sftp-service** | Azure VM | ✅ Deployed | [View](./klubben-sftp-service-INFRASTRUCTURE_GUIDE.md) |

---

## How to Use This Repository

1. **Find the guide** for your project in the list above
2. **Read the relevant section** (deployment, troubleshooting, monitoring, etc.)
3. **Follow the procedures** step-by-step
4. **Refer to Quick Reference** sections for commands and URLs

Each guide is self-contained and includes:
- Architecture overview
- File structure and locations
- Environment variables and secrets management
- Deployment procedures
- Monitoring and logs
- Testing procedures
- Troubleshooting common issues
- Quick reference (commands, URLs, env vars)

---

## Key Principles

✅ **No secrets exposed** - Guides reference secrets in Azure Key Vault, not expose them  
✅ **Single source of truth** - All procedures documented in one place  
✅ **Version controlled** - Git history tracks changes to procedures  
✅ **Accessible to all** - Public repo (no sensitive data)  
✅ **Easy to search** - Markdown format, indexed by project  

---

## Contributing

When you make infrastructure changes:
1. Update the relevant guide
2. Commit with co-author attribution:
   ```
   Co-Authored-By: Oz <oz-agent@warp.dev>
   ```
3. Push to `main`
4. Keep procedures synchronized across code changes

---

## Cloud & Infrastructure Resources

### Azure Key Vault
- **Purpose**: Centralized secret management
- **Access**: https://portal.azure.com
- **Secrets Stored**: API keys, tokens, credentials

### GitHub
- **Organization**: https://github.com/robinpayman
- **This Repo**: https://github.com/robinpayman/infrastructure-guides
- **All Projects**: Listed above

### Vercel
- **Dashboard**: https://vercel.com
- **Projects**: entrynor-website, wc-tripletex-integration

### Azure VMs
- **SSH Access**: Via SSH keys in Azure Key Vault
- **Services**: PDF integration, SFTP service

---

## Index by Topic

### Deployment
- [Entrynor Deployment](./entrynor-website-INFRASTRUCTURE_GUIDE.md#9-deployment-process)

### Email & Communications
- [Entrynor Contact Form](./entrynor-website-INFRASTRUCTURE_GUIDE.md#4-contact-form-infrastructure)

### Monitoring & Logs
- [Entrynor Monitoring](./entrynor-website-INFRASTRUCTURE_GUIDE.md#10-monitoring--logs)

### Troubleshooting
- [Entrynor Troubleshooting](./entrynor-website-INFRASTRUCTURE_GUIDE.md#13-troubleshooting)

### Secrets & Credentials
- [Entrynor Secrets](./entrynor-website-INFRASTRUCTURE_GUIDE.md#6-password--credential-management)

---

**Last Updated**: August 2026  
**Maintainer**: Robin Payman  
**Backup Location**: Azure Key Vault + OneDrive
