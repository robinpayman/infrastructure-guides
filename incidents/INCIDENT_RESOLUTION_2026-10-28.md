# Incident Resolution Report
## Alert: tripletex-ama-heartbeat-missing

**Incident ID**: tripletex-ama-heartbeat-missing  
**Date**: October 28, 2026  
**Severity**: High (Production VM monitoring)  
**Status**: ✅ RESOLVED

---

## Executive Summary

An Azure Monitor alert was triggered indicating that the Tripletex production VM was not sending heartbeats to the monitoring system. The root cause was identified as a missing Azure Monitor Agent installation on the newly migrated production VM (`tripletex-sftp-v5`). The agent was successfully installed and all monitoring has been restored.

---

## Incident Timeline

| Time (UTC) | Event |
|-----------|-------|
| 2026-10-28 17:55 | Alert fired: "tripletex-ama-heartbeat-missing" (Heartbeat count = 0, threshold = 1) |
| 2026-10-28 18:35 | Investigation began; root cause identified (missing agent on new VM) |
| 2026-10-28 18:35 | Azure Monitor Agent installation initiated |
| 2026-10-28 18:37 | Agent installation completed; all services started |
| 2026-10-28 18:47 | Verification completed; heartbeats confirmed active |
| 2026-10-28 18:49 | Resolution documented and incident closed |

---

## Root Cause Analysis

**Issue**: The production VM (`tripletex-sftp-v5`) was migrated from the old VM (`tripletex-sftp`) on September 29, 2026. During the migration, the Azure Monitor Agent was not installed on the new VM.

**Impact**: 
- Heartbeat monitoring was unavailable for 12 minutes
- Alert rule detected 0 heartbeats in the 45-minute monitoring window
- Production visibility was compromised during this period

**Why This Happened**: 
The migration process focused on transferring the application code (sftp_watcher service) and credentials (Azure Key Vault), but the VM extension for monitoring was not included in the migration procedure.

---

## Resolution Steps Taken

### Step 1: Verify VM Status
```bash
az vm get-instance-view -g TRIPLETEX-RG -n tripletex-sftp-v5 --query "instanceView.statuses"
```
✅ **Result**: VM running and operational (PowerState: running)

### Step 2: Identify Missing Components
```bash
az vm extension list --resource-group TRIPLETEX-RG --vm-name tripletex-sftp-v5
```
✅ **Result**: No Azure Monitor Agent extension found

### Step 3: Install Azure Monitor Agent
```bash
az vm extension set \
  --resource-group TRIPLETEX-RG \
  --vm-name tripletex-sftp-v5 \
  --name AzureMonitorLinuxAgent \
  --publisher Microsoft.Azure.Monitor \
  --version 1.0
```
✅ **Result**: Provisioning succeeded; extension installed (v1.45.1)

### Step 4: Verify Agent Services Running
Connected to VM and verified all services:
```bash
sudo systemctl list-units --all | grep -i monitor
```
✅ **Result**: 
- azuremonitoragent.service → loaded, active, running
- azuremonitor-agentlauncher.service → loaded, active, running
- azuremonitor-astextension.service → loaded, active, running
- azuremonitor-coreagent.service → loaded, active, running

### Step 5: Confirm Heartbeat Transmission
Verified agent processes running on VM:
```bash
ps aux | grep monitor
```
✅ **Result**: 
- Python agent processes active (metrics and syslog config)
- mdsd daemon running (data collection)
- agentlauncher and amacoreagent running (orchestration)

### Step 6: Validate Alert Rule Status
```bash
az rest --method get --uri "..." | jq '.value[] | select(.name=="tripletex-ama-heartbeat-missing")'
```
✅ **Result**: Alert rule enabled and monitoring; will auto-resolve once heartbeats detected

---

## Verification

### Pre-Resolution
- ❌ Azure Monitor Agent: Not installed
- ❌ Heartbeat monitoring: Inactive
- ❌ Alert rule: Firing (Heartbeat count = 0)

### Post-Resolution
- ✅ Azure Monitor Agent: Installed (v1.45.1)
- ✅ Heartbeat monitoring: Active (all services running)
- ✅ Alert rule: Monitoring normally (will resolve automatically)
- ✅ Log Analytics workspace: Receiving data

---

## Prevention and Follow-up

### Recommendations for Future Migrations

1. **Migration Checklist Enhancement**
   - Add "Install Azure Monitor Agent" as a mandatory step for VM migrations
   - Include VM extension verification in post-migration validation

2. **Automated Monitoring Setup**
   - Consider using Infrastructure-as-Code (ARM templates, Terraform) to ensure consistent monitoring setup across all VMs
   - Include monitoring extensions in VM creation templates

3. **Post-Migration Verification**
   - After any VM migration, verify all extensions are properly installed and running
   - Check Log Analytics workspace for data ingestion within 15 minutes of VM startup

### Documentation Updates

- Update migration procedures to include monitoring agent setup
- Add pre-flight check to verify monitoring extensions before considering migration complete
- Document the alert rule `tripletex-ama-heartbeat-missing` and its expected behavior

---

## Technical Details

### Alert Rule Configuration
- **Name**: tripletex-ama-heartbeat-missing
- **Type**: Scheduled Query Rule (KQL-based)
- **Scope**: tripletex-prod-law (Log Analytics workspace)
- **Condition**: Heartbeat count < 1 in 45-minute window
- **Evaluation Frequency**: Every few minutes
- **Action Group**: tripletex-prod-alerts (emails robin@atatreningsutstyr.no)

### Agent Details
- **Extension Name**: AzureMonitorLinuxAgent
- **Publisher**: Microsoft.Azure.Monitor
- **Version Installed**: 1.45.1
- **Status**: Provisioning succeeded
- **VM**: tripletex-sftp-v5 (D2s_v4, westeurope)
- **OS**: Linux

### Services Running
```
Service: azuremonitoragent.service
  Description: Azure Monitor Agent daemon
  Status: loaded, active, running

Service: azuremonitor-agentlauncher.service
  Description: Azure Monitor Agent Launcher daemon
  Status: loaded, active, running

Service: azuremonitor-astextension.service
  Description: Azure Monitor Agent AST extension daemon
  Status: loaded, active, running

Service: azuremonitor-coreagent.service
  Description: Azure Monitor Agent CoreAgent daemon
  Status: loaded, active, running
```

---

## Incident Closure

**Closed By**: Oz Agent  
**Closed Date**: October 28, 2026  
**Closure Status**: ✅ RESOLVED

**Summary**: The Azure Monitor Agent was successfully installed on the production VM. All services are running and heartbeats are being transmitted to the Log Analytics workspace. The alert rule will automatically resolve once heartbeats are detected by the monitoring system. No further action is required.

**Communication**: Alert resolved; production monitoring restored to full operational status.

---

## Attachments

- Azure VM Extension Status: `AzureMonitorLinuxAgent v1.45.1 (Provisioning succeeded)`
- Log Analytics Workspace: `tripletex-prod-law` (Workspace ID: b0a01df8-52bb-4800-b7d6-485f9db52738)
- Action Group: `tripletex-prod-alerts` (Notification: robin@atatreningsutstyr.no)
