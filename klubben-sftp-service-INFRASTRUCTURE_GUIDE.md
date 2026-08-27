# Klubben SFTP Service Infrastructure Guide

Complete reference for the Klubben SFTP file processing and Tripletex synchronization service.

---

## 1. Project Overview

**Project Name**: Klubben SFTP Service  
**Repository**: https://github.com/robinpayman/klubben-sftp-service  
**Deployment**: Azure VM (Linux)  
**Type**: Python SFTP watcher service (similar to tripletex-pdf-integration)  
**Purpose**: Monitor SFTP folder for Klubben XML order files, validate data, and sync with Tripletex  

### Key Features
- **SFTP Monitoring**: Watch incoming folder for new Klubben XML files
- **Order Processing**: Parse and validate Klubben XML order format
- **Tripletex Sync**: Create orders in Tripletex from Klubben data
- **Customer Mapping**: Link Klubben customers to Tripletex
- **Error Handling**: Move failed files to /failed/ folder
- **Duplicate Detection**: Reference-based duplicate detection
- **24/7 Operation**: systemd service with auto-restart
- **Email Alerts**: Notify on processing failures

---

## 2. Service Architecture

### Similar to tripletex-pdf-integration

**Key Difference**: Instead of PDF extraction, this service:
1. Reads Klubben XML files from SFTP
2. Parses XML order structure
3. Maps Klubben fields to Tripletex format
4. Creates orders in Tripletex

### File Format

**Input**: Klubben XML files  
**Reference Field**: "Bestillingsnr" (Order number) per Rule #M1nZruurzYXH75wvq4CeqB  
**Output**: Tripletex orders

---

## 3. Deployment

### Local to Production

Follows same deployment process as **tripletex-pdf-integration**:

```bash
# 1. Local development
git clone https://github.com/robinpayman/klubben-sftp-service.git
cd klubben-sftp-service

# 2. Test locally
python sftp_watcher.py --test

# 3. Commit and push
git push origin main

# 4. SSH into Azure VM
ssh -i ~/.ssh/klubben-sftp-key klubben_user@<vm-ip>

# 5. Deploy
cd /opt/klubben/app
git pull origin main
sudo systemctl restart klubben_sftp_watcher.service
```

---

## 4. SFTP Folder Structure

**Root Path**: `/sftp/customers/klubben/`

```
/sftp/customers/klubben/
├── pending/       - Incoming Klubben XML files
├── completed/     - Successfully processed files
├── failed/        - Files that failed validation
├── duplicates/    - Duplicate orders by reference
└── logs/          - Processing logs
```

---

## 5. Key Configuration

### Service Name

`klubben_sftp_watcher.service`

### Environment Variables

Similar to tripletex-pdf-integration:
- `TRIPLETEX_CONSUMER_TOKEN` - API identification
- `TRIPLETEX_EMPLOYEE_TOKEN` - User authentication
- `KLUBBEN_REFERENCE_FIELD` - XML field for order number ("Bestillingsnr")
- Email configuration for failure notifications

### SSH Key

`~/.ssh/klubben-sftp-key` (permissions: 600)

---

## 6. Monitoring & Logs

### Service Status

```bash
ssh -i ~/.ssh/klubben-sftp-key klubben_user@<vm-ip> \
  "sudo systemctl status klubben_sftp_watcher.service"
```

### Real-Time Logs

```bash
ssh -i ~/.ssh/klubben-sftp-key klubben_user@<vm-ip> \
  "sudo journalctl -u klubben_sftp_watcher.service -f"
```

### Check Files Processing

```bash
# List pending files
ls -la /sftp/customers/klubben/pending/

# List completed files
ls -la /sftp/customers/klubben/completed/

# List failed files
ls -la /sftp/customers/klubben/failed/
```

---

## 7. Troubleshooting

### Service Not Running

```bash
ssh -i ~/.ssh/klubben-sftp-key klubben_user@<vm-ip>
sudo systemctl restart klubben_sftp_watcher.service
sudo journalctl -u klubben_sftp_watcher.service -n 20
```

### Files Not Processing

1. Check service is running: `sudo systemctl is-active klubben_sftp_watcher.service`
2. Check file in /pending/: `ls -la /sftp/customers/klubben/pending/`
3. Check XML format validity
4. View logs for validation errors

### API Errors

Same troubleshooting as WC-Tripletex Integration:
- Verify Tripletex tokens are current
- Check customer mapping in Tripletex
- Ensure products are defined in Tripletex
- Validate order amounts

---

## 8. Deployment Checklist

Before going live:

- [ ] Klubben XML format is fully documented
- [ ] Field mapping to Tripletex is tested
- [ ] Customer mapping rules are implemented
- [ ] Error handling for invalid XML
- [ ] Duplicate detection by "Bestillingsnr"
- [ ] Email alerts configured
- [ ] Service auto-restart enabled
- [ ] Logs are being written
- [ ] Daily backups configured
- [ ] Team trained on monitoring

---

## 9. Quick Reference

### Important Commands

```bash
# Check service status
sudo systemctl status klubben_sftp_watcher.service

# Restart service
sudo systemctl restart klubben_sftp_watcher.service

# View logs
sudo journalctl -u klubben_sftp_watcher.service -f

# File management
ls /sftp/customers/klubben/pending/
ls /sftp/customers/klubben/completed/
ls /sftp/customers/klubben/failed/
```

### SSH/SFTP Access

```bash
# SSH into VM
ssh -i ~/.ssh/klubben-sftp-key klubben_user@<vm-ip>

# SFTP access
sftp -i ~/.ssh/klubben-sftp-key klubben_user@<vm-ip>
```

### Paths

| Component | Path |
| --- | --- |
| App Directory | `/opt/klubben/app/` |
| Config | `/opt/klubben/.env.production` |
| SFTP Root | `/sftp/customers/klubben/` |
| Service File | `/etc/systemd/system/klubben_sftp_watcher.service` |
| Logs | `journalctl -u klubben_sftp_watcher.service` |

---

## 10. Full Documentation

For complete infrastructure details (similar to Tripletex PDF Integration):
- See: **tripletex-pdf-integration-INFRASTRUCTURE_GUIDE.md**
- The architecture, monitoring, and troubleshooting is nearly identical
- Main difference: XML parsing vs. PDF extraction

Key sections from the PDF guide that apply here:
- Section 3: Production Hosting (Azure VM setup)
- Section 6: Environment Variables & Secrets
- Section 8: Monitoring & Logs
- Section 9: Troubleshooting
- Section 14: Quick Reference
- Section 15: Disaster Recovery
- Section 16: Klubben Order Invoicing (Cross-Reference) - order **creation** only happens here; invoicing (with a 30% shipping-cost safety check) is handled separately by `wc-tripletex-integration` - see that project's infrastructure guide, Section 17.

---

**Last Updated**: August 2026  
**Maintainer**: Robin Payman  
**Status**: Planned/In Development  
**Backup Location**: Azure Key Vault + OneDrive

**Note**: Complete detailed guide following tripletex-pdf-integration model to be expanded once deployment is finalized.
