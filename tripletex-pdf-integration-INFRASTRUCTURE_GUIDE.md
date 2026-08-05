# Tripletex PDF Integration Infrastructure Guide

Complete reference for the PDF file processing and Tripletex synchronization service on Azure.

---

## 1. Project Overview

**Project Name**: Tripletex PDF Integration  
**Repository**: https://github.com/robinpayman/tripletex-pdf-integration  
**Deployment**: Azure VM (Linux)  
**Type**: Python SFTP watcher service  
**Purpose**: Monitor SFTP folder for PDF invoices, validate data, and sync with Tripletex  

### Key Features
- **SFTP Monitoring**: Watch incoming folder for new PDF files
- **Invoice Processing**: Extract and validate invoice data from PDFs
- **Tripletex Sync**: Create/update invoices in Tripletex
- **Error Handling**: Move failed files to /failed/ folder
- **Duplicate Detection**: Hash-based duplicate detection
- **24/7 Operation**: systemd service with auto-restart
- **Logging**: Comprehensive error and activity logs

---

## 2. Code Repository

### GitHub

**Repository**: `robinpayman/tripletex-pdf-integration`  
**URL**: https://github.com/robinpayman/tripletex-pdf-integration  
**Main Branch**: `main`  
**Access**: SSH via GitHub  

### Local Development

**Directory**: `/Users/rp/projects/tripletex-pdf-integration`  
**Type**: Python (Flask/Gunicorn)  
**Key Files**:
- `sftp_watcher.py` - Main service that monitors SFTP folder
- `config_production.py` - Production configuration
- `requirements.txt` - Python dependencies
- `deploy.sh` - Deployment script

---

## 3. Production Hosting

### Azure VM Details

**VM Name**: tripletex-sftp  
**OS**: Linux (Ubuntu)  
**Region**: Azure cloud region  
**Access**: SSH via private key  
**Service User**: `tripletex_user` (non-root)  

### Service Configuration

**Service Name**: `sftp_watcher.service`  
**Service File**: `/etc/systemd/system/sftp_watcher.service`  
**App Path**: `/opt/tripletex/app/`  
**Log Location**: journalctl logs  

**Service Properties**:
- `Restart=always` - Auto-restart on failure
- `RestartSec=10` - Restart after 10 seconds on failure
- `Type=simple` - Simple service type
- `Enabled=true` - Boot startup enabled

### SFTP Configuration

**SFTP Server**: Same Azure VM (built-in SFTP)  
**Root Folder**: `/sftp/customers/tripletex/`  
**Subfolder Structure**:
```
/sftp/customers/tripletex/
├── pending/       - Incoming PDF files to process
├── completed/     - Successfully processed files
├── failed/        - Files that failed validation
├── duplicates/    - Duplicate files (by hash match)
└── logs/          - Processing logs
```

**Users**: Read/write access via SSH keys stored in Azure Key Vault

---

## 4. Deployment Architecture

### Workflow

**File Flow**:
```
1. PDF uploaded to /pending/ via SFTP
2. sftp_watcher.py detects new file
3. Validates file hash (duplicate check)
4. Extracts invoice data from PDF
5. Validates data completeness
6. Creates invoice in Tripletex API
7. Moves file to /completed/ or /failed/
8. Logs result
9. Sends email on failure (if enabled)
```

### File Movements

**Success Flow**:
- `pending/invoice.pdf` → `completed/invoice.pdf`

**Failure Flow**:
- `pending/invoice.pdf` → `failed/invoice.pdf` (validation error)
- `pending/invoice.pdf` → `duplicates/invoice.pdf` (hash match)

---

## 5. Deployment Process

### Local to Azure

**Step 1**: Make changes locally in `/Users/rp/projects/tripletex-pdf-integration`  
**Step 2**: Test locally:
```bash
python sftp_watcher.py --test
```

**Step 3**: Commit and push to GitHub:
```bash
git add .
git commit -m "Your message

Co-Authored-By: Oz <oz-agent@warp.dev>"
git push origin main
```

**Step 4**: Deploy to Azure:
```bash
# SSH into Azure VM
ssh -i ~/.ssh/tripletex-sftp-key tripletex_user@<vm-ip>

# Navigate to app directory
cd /opt/tripletex/app

# Pull latest code
git pull origin main

# Verify commit hash matches GitHub
git log -1 --oneline

# Restart service
sudo systemctl restart sftp_watcher.service

# Check status
sudo systemctl status sftp_watcher.service

# View logs
sudo journalctl -u sftp_watcher.service -f
```

**Step 5**: Verify deployment:
```bash
# Check service is running
sudo systemctl is-active sftp_watcher.service

# Watch logs for activity
sudo journalctl -u sftp_watcher.service -n 50

# Test SFTP access
sftp -i ~/.ssh/tripletex-sftp-key tripletex_user@<vm-ip>
```

---

## 6. Environment Variables & Secrets

### Production (Azure VM)

**File**: `/opt/tripletex/.env.production`  
**Access**: SSH only  
**Sensitive**: Contains API tokens  

```bash
# Tripletex API Configuration
TRIPLETEX_CONSUMER_TOKEN=your-consumer-token
TRIPLETEX_EMPLOYEE_TOKEN=your-employee-token
TRIPLETEX_API_URL=https://api.tripletex.io

# Email Configuration
EMAIL_ENABLED=true
EMAIL_METHOD=graph  # or smtp
EMAIL_GRAPH_TENANT_ID=your-tenant-id
EMAIL_GRAPH_CLIENT_ID=your-client-id
EMAIL_GRAPH_CLIENT_SECRET=your-secret
EMAIL_GRAPH_SENDER=notify@example.com
EMAIL_TO_FAILURE=robin@atatreningsutstyr.no

# SFTP Configuration
SFTP_HOST=localhost
SFTP_PORT=22
SFTP_USER=tripletex_user
SFTP_PRIVATE_KEY=/home/tripletex_user/.ssh/id_rsa
SFTP_ROOT=/sftp/customers/tripletex/

# Logging
LOG_LEVEL=INFO
LOG_FILE=/opt/tripletex/logs/sftp_watcher.log
```

### Local Development

**File**: `config_production.py`  
**Contains**: Settings for testing (can reference Azure Key Vault)

---

## 7. Password & Credential Management

### Azure Key Vault (Preferred)

**Location**: Azure Portal  
**Secrets Stored**:
- Tripletex API tokens (PRODUCTION and TEST)
- SFTP private key (tripletex-sftp-key)
- Email service credentials (Azure Graph API)
- GitHub personal access token

### Local Backup

**File**: `.credentials-database.md`  
**Location**: `/Users/rp/projects/tripletex-pdf-integration/.credentials-database.md`  
**Sync**: Backup to OneDrive within 24 hours of changes  
**Format**: Operational references only (no actual secrets)

### SSH Key Management

**Key File**: `~/.ssh/tripletex-sftp-key`  
**Permissions**: `600` (owner read/write only)  
**Backup**: Azure Key Vault + OneDrive  
**Scope**: SSH access to Azure VM

**Verify Permissions**:
```bash
ls -l ~/.ssh/tripletex-sftp-key
# Should show: -rw------- (600)
```

### Email Configuration

**Service**: Microsoft Graph API (preferred)  
**Alternative**: SMTP (less secure)  
**Credentials**: Stored in Azure Key Vault  
**Test**:
```bash
# SSH into VM and test email
python -c "from app import send_email; send_email('test@example.com', 'Test', 'Test message')"
```

---

## 8. Monitoring & Logs

### Service Status

**Check Service**:
```bash
sudo systemctl status sftp_watcher.service
```

**Expected Output**:
```
● sftp_watcher.service - Tripletex PDF SFTP Watcher
   Loaded: loaded (/etc/systemd/system/sftp_watcher.service; enabled; vendor preset: enabled)
   Active: active (running) since ...
   Main PID: 1234
```

### Real-Time Logs

**Watch Logs**:
```bash
ssh -i ~/.ssh/tripletex-sftp-key tripletex_user@<vm-ip> \
  "sudo journalctl -u sftp_watcher.service -f"
```

**What to Look For**:
- File detection: "Detected new PDF: invoice.pdf"
- Validation: "Validating invoice.pdf"
- Success: "Successfully created order X in Tripletex"
- Failure: "Failed to process: validation error"
- Duplicates: "Moved duplicate to duplicates folder"

### Recent Logs

**View Last 50 Lines**:
```bash
sudo journalctl -u sftp_watcher.service -n 50
```

**Search Logs**:
```bash
sudo journalctl -u sftp_watcher.service --grep="ERROR"
sudo journalctl -u sftp_watcher.service --since="2 hours ago"
```

### File System Logs

**Application Log** (if file logging enabled):
```bash
tail -f /opt/tripletex/logs/sftp_watcher.log
```

---

## 9. Troubleshooting

### Service Won't Start

**Check**:
1. Service file exists: `sudo systemctl cat sftp_watcher.service`
2. Config file valid: `python -m py_compile config_production.py`
3. Dependencies installed: `pip list | grep -E 'flask|gunicorn'`
4. Permissions: `ls -l /opt/tripletex/app/`

**Restart**:
```bash
sudo systemctl restart sftp_watcher.service
sudo journalctl -u sftp_watcher.service -n 20  # Check errors
```

### SFTP Connection Failures

**Check**:
1. SFTP server running: `sudo systemctl status ssh`
2. Key permissions: `ls -l ~/.ssh/tripletex-sftp-key` (should be 600)
3. Key registered: Check SSH public key in Azure VM
4. Network: `ping <vm-ip>`

**Test Connection**:
```bash
ssh -i ~/.ssh/tripletex-sftp-key tripletex_user@<vm-ip> "whoami"
```

### Files Not Processing

**Check**:
1. Service is running: `sudo systemctl is-active sftp_watcher.service`
2. File is in /pending/: `ls -la /sftp/customers/tripletex/pending/`
3. File permissions: `ls -la` (should be readable by service user)
4. Logs for errors: `sudo journalctl -u sftp_watcher.service -f`

**Common Issues**:
- **File locked**: Another process reading file, wait a moment
- **PDF invalid**: PDF format error, move to /failed/ manually
- **Tripletex API error**: Check token, network, API status
- **Duplicate detected**: File already processed (hash match)

### Email Not Sending

**Check**:
1. EMAIL_ENABLED=true in .env.production
2. Email credentials valid in Azure Key Vault
3. Recipient email is correct
4. Check logs for email errors: `grep -i email journalctl`

**Test Email Manually**:
```bash
ssh -i ~/.ssh/tripletex-sftp-key tripletex_user@<vm-ip>
cd /opt/tripletex/app
python -c "from sftp_watcher import send_error_email; send_error_email('test@example.com', 'Test Subject', 'Test message')"
```

### Disk Space Issues

**Check Disk Usage**:
```bash
df -h /opt/tripletex
df -h /sftp/customers/tripletex
```

**Clean Up Old Files**:
```bash
# Archive completed files older than 30 days
find /sftp/customers/tripletex/completed -mtime +30 -tar -czf archive.tar.gz
```

---

## 10. API Integration

### Tripletex Invoice Creation

**Endpoint**: `POST /v2/accounting/order`  
**Authentication**: Bearer token (consumer + employee)  
**Required Fields**:
- Order number (WooCommerce order ID or unique identifier)
- Customer (must exist in Tripletex or be created)
- Order lines (products and amounts)
- Order date
- Total amount (must be validated)

**Validation Before Sync**:
- Customer exists or can be created
- All products are mapped
- Total amount verified
- Required fields present

### Error Handling

**Validation Errors** (don't retry):
- Missing customer: Create customer in Tripletex
- Missing product: Add product to Tripletex
- Amount mismatch: Fix in source system

**Transient Errors** (retry automatically):
- Network timeout: Retried with exponential backoff
- Rate limiting (429): Retried with delay
- Temporary server error (5xx): Retried

---

## 11. Testing the Service

### Local Testing

```bash
# Run in test mode
python sftp_watcher.py --test

# Test PDF validation
python -c "from pdf_processor import extract_data; extract_data('sample.pdf')"

# Test Tripletex API connection
curl -H "Authorization: Bearer $TRIPLETEX_TOKEN" \
  https://api.tripletex.io/v2/accounting/order
```

### Production Testing

**1. Test File Upload**:
```bash
# Upload test PDF to pending folder
sftp -i ~/.ssh/tripletex-sftp-key tripletex_user@<vm-ip>
put sample.pdf /pending/

# Monitor logs
sudo journalctl -u sftp_watcher.service -f
```

**2. Verify Processing**:
```bash
# Check if file moved to completed
ls -la /sftp/customers/tripletex/completed/

# Verify order in Tripletex
curl -H "Authorization: Bearer $TOKEN" \
  https://api.tripletex.io/v2/accounting/order?number=ORDER_ID
```

**3. Error Handling**:
```bash
# Upload invalid PDF to test error handling
# Check it moves to /failed/
ls -la /sftp/customers/tripletex/failed/

# Verify email sent
# Check email inbox
```

---

## 12. 24/7 Operation

### Service Health Check

**Weekly Status Check**:
```bash
ssh -i ~/.ssh/tripletex-sftp-key tripletex_user@<vm-ip> \
  "sudo systemctl is-active sftp_watcher.service && echo 'RUNNING' || echo 'STOPPED'"
```

**Monthly Log Review**:
```bash
# Check for persistent errors
sudo journalctl -u sftp_watcher.service --since="30 days ago" | grep -i error | wc -l
```

### Alerts & Monitoring

**Recommended Alerts**:
- Service crashes: Notify via email
- Processing failures: Daily digest
- Disk full: Immediate alert
- No files processed (24h): Check logs

**Setup Monitoring**:
```bash
# Create monitoring script
cat > /opt/tripletex/monitor.sh << 'EOF'
#!/bin/bash
if ! sudo systemctl is-active sftp_watcher.service > /dev/null; then
  # Send alert email
  mail -s "Service Down: sftp_watcher" robin@example.com <<< "Service is not running"
  sudo systemctl restart sftp_watcher.service
fi
EOF

# Add to crontab (every 5 minutes)
*/5 * * * * /opt/tripletex/monitor.sh
```

---

## 13. Maintenance Tasks

### Regular Tasks

**Daily**:
- Monitor logs for errors
- Check SFTP folder size
- Verify service is running

**Weekly**:
- Verify completed files being processed
- Check for stuck files in pending/
- Review error log for patterns

**Monthly**:
- Archive old completed files
- Review and optimize error handling
- Test disaster recovery (restore from backup)

### Backup Strategy

**What to Backup**:
- `/opt/tripletex/app` (code)
- `/opt/tripletex/.env.production` (config)
- `/sftp/customers/tripletex/completed` (processed files)
- Service config: `/etc/systemd/system/sftp_watcher.service`

**Backup Command**:
```bash
tar -czf tripletex_backup_$(date +%Y%m%d).tar.gz \
  /opt/tripletex \
  /etc/systemd/system/sftp_watcher.service
```

**Restore**:
```bash
tar -xzf tripletex_backup_YYYYMMDD.tar.gz -C /
sudo systemctl daemon-reload
sudo systemctl restart sftp_watcher.service
```

---

## 14. Quick Reference

### Important URLs & Paths

| Resource | Location |
| --- | --- |
| GitHub Repo | https://github.com/robinpayman/tripletex-pdf-integration |
| Azure VM | <vm-ip-address> |
| App Directory | `/opt/tripletex/app/` |
| Config File | `/opt/tripletex/.env.production` |
| SFTP Root | `/sftp/customers/tripletex/` |
| Service File | `/etc/systemd/system/sftp_watcher.service` |
| Logs | `journalctl -u sftp_watcher.service` |

### SSH/SFTP Access

```bash
# SSH into Azure VM
ssh -i ~/.ssh/tripletex-sftp-key tripletex_user@<vm-ip>

# SFTP access
sftp -i ~/.ssh/tripletex-sftp-key tripletex_user@<vm-ip>

# Remote command execution
ssh -i ~/.ssh/tripletex-sftp-key tripletex_user@<vm-ip> "sudo systemctl status sftp_watcher.service"
```

### Command Cheat Sheet

```bash
# Service management
sudo systemctl status sftp_watcher.service     # Check status
sudo systemctl restart sftp_watcher.service    # Restart
sudo systemctl stop sftp_watcher.service       # Stop
sudo systemctl start sftp_watcher.service      # Start
sudo systemctl enable sftp_watcher.service     # Enable on boot

# Logging
sudo journalctl -u sftp_watcher.service -f     # Follow logs
sudo journalctl -u sftp_watcher.service -n 50  # Last 50 lines
sudo journalctl -u sftp_watcher.service --since="1 hour ago"  # Recent logs

# File management
ls -la /sftp/customers/tripletex/pending/      # Pending files
ls -la /sftp/customers/tripletex/completed/    # Processed files
ls -la /sftp/customers/tripletex/failed/       # Failed files
du -sh /sftp/customers/tripletex/              # Folder size

# Deployment
cd /opt/tripletex/app && git pull origin main  # Update code
cd /opt/tripletex/app && git log -1 --oneline  # Verify commit
```

### Environment Variables

| Variable | Sensitivity | Purpose |
| --- | --- | --- |
| `TRIPLETEX_CONSUMER_TOKEN` | **SECRET** | App identification |
| `TRIPLETEX_EMPLOYEE_TOKEN` | **SECRET** | User authentication |
| `EMAIL_GRAPH_CLIENT_SECRET` | **SECRET** | Email service auth |
| `SFTP_PRIVATE_KEY` | **SECRET** | SFTP access |
| `TRIPLETEX_API_URL` | Public | API endpoint |
| `EMAIL_TO_FAILURE` | Sensitive | Alert recipient |

---

## 15. Disaster Recovery

### Service Down (Unresponsive)

```bash
# 1. SSH into VM
ssh -i ~/.ssh/tripletex-sftp-key tripletex_user@<vm-ip>

# 2. Restart service
sudo systemctl restart sftp_watcher.service

# 3. Monitor logs
sudo journalctl -u sftp_watcher.service -f

# 4. If still failing, check disk/memory
df -h
free -h

# 5. Check recent changes
cd /opt/tripletex/app && git log -3 --oneline
git diff HEAD~1 HEAD

# 6. Rollback if needed
git reset --hard HEAD~1
sudo systemctl restart sftp_watcher.service
```

### Data Loss (Completed Files)

```bash
# 1. Check backups
ls -la /sftp/backups/

# 2. Restore from backup
tar -xzf /sftp/backups/completed_files_backup.tar.gz -C /sftp/customers/tripletex/

# 3. Verify integrity
du -sh /sftp/customers/tripletex/completed/
find /sftp/customers/tripletex/completed/ -type f | wc -l
```

### Full VM Recovery

```bash
# 1. Verify backup is current
ls -lh tripletex_backup_latest.tar.gz

# 2. Create new VM from image

# 3. Deploy app
cd /opt && tar -xzf tripletex_backup_latest.tar.gz

# 4. Start service
sudo systemctl daemon-reload
sudo systemctl start sftp_watcher.service

# 5. Monitor
sudo journalctl -u sftp_watcher.service -f
```

---

**Last Updated**: August 2026  
**Maintainer**: Robin Payman  
**Status**: Production  
**Backup Location**: Azure Key Vault + OneDrive
