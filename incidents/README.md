# Infrastructure Incidents Archive

This directory contains documented incident resolutions for infrastructure and operations.

## Purpose

- Maintain historical records of production incidents
- Enable root cause analysis and learning
- Support compliance and audit requirements
- Facilitate team training and knowledge sharing

## Incidents

### 2026-10-28: Azure Monitor Heartbeat Alert

**File**: `INCIDENT_RESOLUTION_2026-10-28.md`

**Summary**:
- Alert: `tripletex-ama-heartbeat-missing`
- Severity: High (Production)
- Duration: 54 minutes
- Root Cause: Missing Azure Monitor Agent on new VM post-migration
- Resolution: Installed AzureMonitorLinuxAgent v1.45.1
- Status: ✅ RESOLVED

**Key Files**:
- `INCIDENT_RESOLUTION_2026-10-28.md` - Full incident report
- `NOTIFICATION_2026-10-28.txt` - Team notification email

---

## Using This Archive

1. **For Incident Investigation**: Review the full incident report for root cause analysis
2. **For Prevention**: Check the "Prevention Measures" section for recommendations
3. **For Training**: Use as educational material for team training
4. **For Compliance**: Archive records support compliance requirements
5. **For Procedures**: Update operational procedures based on recommendations

## Adding New Incidents

When documenting a new incident:

1. Create a detailed incident report with:
   - Executive summary
   - Timeline of events
   - Root cause analysis
   - Resolution steps taken
   - Verification results
   - Prevention measures
   - Follow-up actions

2. Include supporting files:
   - Team notification email
   - Any relevant logs or screenshots
   - Technical specifications

3. Add entry to this README

4. Commit to repository with message:
   ```
   docs(incidents): add [INCIDENT_ID] resolution report
   
   Summary: [Brief description]
   Duration: [X minutes/hours]
   Root Cause: [Summary]
   Status: RESOLVED
   
   Co-Authored-By: Oz <oz-agent@warp.dev>
   ```

---

**Last Updated**: October 28, 2026
**Archive Maintainer**: Infrastructure Team
