# Plan for Windows 365 Day 2 operations

[!INCLUDE [applies-to-windows-365-enterprise](includes/applies-to-windows-365-enterprise-and-frontline.md)]

This guide provides IT administrators with structured guidance for ongoing operational management of Windows 365 Cloud PCs after initial deployment. Day 2 operations encompass monitoring, alerting, recovery, patching, security maintenance, and continuous optimization—the activities required to maintain a healthy, performant, and resilient Cloud PC environment.

## Overview

After deploying Windows 365 (Day 1), administrators shift focus to ongoing operations. Day 2 operations ensure that Cloud PCs remain available, secure, and performant while minimizing user disruption. This guide covers:

| Area | Purpose |
|------|---------|
| [Monitoring and reporting](#monitoring-and-reporting) | Maintain visibility into Cloud PC health and performance |
| [Alerting](#alerting) | Proactively detect and respond to issues before user impact |
| [Business continuity and disaster recovery](#business-continuity-and-disaster-recovery) | Protect against data loss and regional outages |
| [Patch and update management](#patch-and-update-management) | Keep Cloud PCs current and secure |
| [Security operations](#security-operations) | Maintain security posture post-deployment |
| [Remote actions and remediation](#remote-actions-and-remediation) | Resolve issues without user intervention |
| [Capacity and cost optimization](#capacity-and-cost-optimization) | Right-size resources and manage licensing |
| [Operational automation](#operational-automation) | Reduce manual effort through automation |

## Monitoring and reporting

Continuous monitoring provides the foundation for all Day 2 operations. Windows 365 provides built-in reporting through the Microsoft Intune admin center and Endpoint Analytics.

### Cloud PC monitoring (preview)

Cloud PC Monitoring provides IT admins with enhanced visibility into Cloud PC health, performance, and configuration through integrated dashboards.

**Access**: Sign in to the [Microsoft Intune admin center](https://go.microsoft.com/fwlink/?linkid=2109431) > **Reports** > **Cloud PC monitoring (preview)**.

The monitoring experience is organized into three tabs:

| Tab | Use case |
|-----|----------|
| **Connection health** | Observe your full Windows 365 tenant, identify when a problem occurs, and isolate affected cohorts |
| **User and devices** | Investigate detailed history for specific users or devices; supports help-desk scenarios |
| **Configuration monitoring** | Understand end-to-end connection configurations aggregated across your environment |

Key connection health metrics include:

- **Active connection count** – Trend of active connections over time
- **Connection failure rate** – Connections experiencing failures
- **Cloud PC health** – Percentage of Cloud PCs passing health validation tests
- **Round trip time (RTT)** – Median P50 round trip time for all Cloud PCs

### Connection quality report

The **Connection quality report** helps identify devices performing below expectations.

**Access**: **Reports** > **Cloud PC overview** > **Connection quality**

Key metrics per Cloud PC:

- Round trip time (most recent and P50 over date range)
- Available bandwidth (most recent and P50)
- Connection gateway
- Remoting sign-in time (P50)
- Protocol and UDP utilization
- Client IP address

Use filters to identify connections that don't meet organizational requirements and evaluate causes such as:

- Network, host, or client configurations limiting protocol choice
- Geographic distance between users and Cloud PCs
- Authentication methods and architecture
- Client or host appropriateness

### Resource performance report

Available through **Reports** > **Endpoint analytics** > **Resource performance**, this report helps optimize vCPU and RAM resources.

Key capabilities:

- **Resource performance score** – Overall rating (0–100) based on CPU and RAM spike time
- **Model performance** – Compare performance across Cloud PC SKUs
- **Device performance** – Per-device CPU and RAM spike percentages
- **Device history** – Trend data for individual Cloud PCs

> [!TIP]
> If a device score falls below baseline, consider a [resize action](https://learn.microsoft.com/windows-365/enterprise/resize-cloud-pc) to add more resources.

### Additional reports

| Report | Purpose |
|--------|---------|
| **Cloud PC utilization** | Hours users are connected; identifies unused Cloud PCs |
| **Cloud PCs that aren't available** | Lists Cloud PCs in an unhealthy state |
| **Connected Frontline Cloud PCs** | Active sessions for Frontline deployments |
| **Cross region disaster recovery status** | Current backup health and readiness |
| **Cloud PC size recommendation** | Recommends appropriate sizes based on usage patterns |

### Copilot in Intune for Windows 365

IT admins can use Copilot in Intune to query Cloud PC information using natural language. Example prompts:

- "Show me my Enterprise Cloud PC licenses"
- "Analyze trends in bandwidth performance"
- "Summarize Cloud PCs that have never been used"

## Alerting

The Windows 365 alert system notifies administrators when specific events occur in the Cloud PC environment. Proactive alerting is critical for resolving issues before they affect end users.

### Built-in alert rules

Windows 365 includes built-in alert rules for the following conditions. Alert severity and thresholds are customizable per rule.

| Alert condition |
|----------------|
| Azure Network Connection failure |
| Provisioning policy failure |
| Custom image upload failure |
| Cloud PCs not available |
| Cloud PCs entering grace period |
| Windows 365 Frontline maximum concurrency reached |
| Pooled user storage running low (User Experience Sync) |

### Configure alert rules

1. Sign in to the [Microsoft Intune admin center](https://go.microsoft.com/fwlink/?linkid=2109431) > **Tenant administration** > **Cloud PC Alerts** > **Alert rules**.
2. Select a rule from the list.
3. On the **System rule** page, configure:
   - **Conditions**: Set thresholds and criteria for triggering
   - **Settings**: Define alert severity and enable/disable the rule
   - **Notifications**: Configure in-console and/or email delivery
4. Select **Apply** to save changes.

### Alert notification channels

| Channel | Behavior |
|---------|----------|
| Console notification | Pop-up alerts in the Intune admin center (enabled by default) |
| Email notification | Sends one email per trigger event to designated addresses; sends again after resolution if re-triggered |

### Recommended alerting strategy

| Priority | Action |
|----------|--------|
| 1 | Enable email notifications for all Critical-severity alert rules |
| 2 | Assign a distribution list (not individual mailboxes) as the alert recipient |
| 3 | Integrate alerts with your IT service management (ITSM) tool via email or the Microsoft Graph API |
| 4 | Review alert rules quarterly to adjust thresholds based on environment growth |
| 5 | Monitor the **Cloud PCs that aren't available** report weekly for emerging patterns |

### View and filter alerts

**Access**: **Tenant administration** > **Cloud PC Alerts**

Filter by:

- **Severity**: Informational, Warning, Critical
- **State**: Active, Resolved

> [!IMPORTANT]
> Alert rules require Windows 365 Enterprise and one of the following roles: Intune Administrator or Windows 365 Administrator.

## Business continuity and disaster recovery

Windows 365 provides multiple layers of data resilience and recovery capabilities. Your BCDR strategy should be tested regularly and aligned with organizational RTO/RPO requirements.

### Shared responsibility model

| Responsibility | Microsoft | Customer |
|----------------|-----------|----------|
| Cloud PC compute availability (99.9% SLA) | ✓ | |
| OS disk durability (11 nines) | ✓ | |
| Automated in-zone compute failover | ✓ | |
| Point-in-time restore configuration | | ✓ |
| Cross region DR licensing and configuration | | ✓ |
| Testing DR activation/deactivation procedures | | ✓ |
| OneDrive/cloud storage configuration for user data | | ✓ |

### Automated in-zone disaster recovery

Windows 365 provides automatic recovery for compute failures within an availability zone:

- **Triggers**: Host failure, power failure, networking issues
- **Behavior**: Workloads automatically move to healthy compute in the same zone
- **RPO**: ~0 (up-to-date copy of OS disk)
- **User impact**: Brief session interruption; user reconnects

No administrator action is required for in-zone recovery.

### Point-in-time restore

Point-in-time restore lets you restore a Cloud PC to the exact state it was at an earlier point in time. Point-in-time restore is always enabled for Windows 365 Enterprise Cloud PCs — restore points are created automatically without any administrator configuration.

#### Default behavior (no user setting required)

| Feature | Behavior |
|---------|----------|
| Short-term restore points | 10 rolling restore points created automatically every 12 hours (default frequency) |
| Long-term restore points | 4 rolling restore points created automatically every 7 days |
| User-initiated restore | Disabled — only administrators can restore Cloud PCs |

#### Configurable via user settings

Administrators can customize restore behavior by creating or editing a user setting in **Devices** > **Windows 365** > **User Settings**:

| Setting | Options | Default |
|---------|---------|---------|
| Short-term restore frequency | Every 4, 6, 12, 16, or 24 hours | 12 hours |
| Allow user to initiate restore service | Yes / No | No |

> [!NOTE]
> Short-term restore points are limited to 10. A shorter frequency provides more granular recovery but covers a shorter total time window. For example, 4-hour intervals cover 40 hours of history, while 24-hour intervals cover 10 days.

#### Manual (on-demand) restore points

Administrators can create a manual restore point for any Cloud PC at any time. Manual restore points are useful before taking administrative actions such as resize or reprovisioning.

| Detail | Value |
|--------|-------|
| Who can create | Administrators only |
| Limit | 1 per Cloud PC (creating a new one overwrites the existing one) |
| Expiration | Approximately 28 days if not overwritten |

#### Best practices

1. Configure the shortest feasible restore interval (4 hours) for critical users
2. Create manual restore points before taking administrative actions (resize, reprovisioning)
3. After restoration, have the user immediately sign in to verify connectivity
4. If a user can't connect post-restore, try a more recent restore point
5. Store critical user data in OneDrive to avoid data loss between restore points

#### Bulk restore

Use **Devices** > **All devices** > **Bulk device actions** to restore multiple Cloud PCs simultaneously from the **Cloud PCs that aren't available** report.

### Cross region disaster recovery

Cross region disaster recovery creates geographically distant copies of Cloud PCs that can be recovered during a regional outage.

| Specification | Value |
|---------------|-------|
| **RTO** | < 4 hours (for up to 50,000 DR-licensed Cloud PCs per backup region) |
| **RPO** | < 4 hours (aligned with point-in-time restore frequency) |
| **Activation** | Manual by administrator |
| **Deactivation** | Manual by administrator or automatic failback 7 days after outage resolution |
| **Capacity** | Best-effort (depends on available capacity in backup region) |

#### Configuration steps

1. **License**: Assign Windows 365 Cross Region Disaster Recovery add-on licenses
2. **User setting**: Create or edit a user setting with cross region DR enabled
3. **Network type**: Select Microsoft-hosted network (recommended) or a pre-configured Azure Network Connection in the backup region
4. **Geography/Region**: Select the backup geography considering data sovereignty and latency
5. **Assign**: Target the user groups requiring DR coverage

> [!IMPORTANT]
> Test your cross region DR procedure regularly. Activate and deactivate test devices to confirm the process works as expected. Use the **Cloud PCs cross region disaster recovery status** report to verify environment health.

### Disaster Recovery Plus

For organizations requiring guaranteed capacity and faster recovery:

| Specification | Cross Region DR | Disaster Recovery Plus |
|---------------|----------------|------------------------|
| **RPO** | < 4 hours | < 61 minutes |
| **RTO** | < 4 hours | < 31 minutes |
| **Capacity reservation** | No (best-effort) | Yes (pre-allocated) |
| **Disk copies** | Snapshots in backup region | Three copies in alternate geography |

### Recovery planning checklist

- [ ] Define RPO/RTO requirements per user group or business function
- [ ] Assign appropriate DR licenses (Cross Region DR or DR Plus) to users meeting defined criteria
- [ ] Configure user settings with backup region and network type
- [ ] Validate initial backup completion using the DR status report (first backup may take several days)
- [ ] Document the activation and deactivation procedure in your runbook
- [ ] Conduct quarterly DR drills, activating and deactivating test devices
- [ ] Confirm OneDrive Known Folder Move is configured to protect user files outside the Cloud PC disk
- [ ] Verify that Azure Network Connection (if used) in the backup region passes health checks

### Service health and outage awareness

Windows 365 depends on multiple Microsoft services (Microsoft Entra ID, Intune, Azure compute, Azure networking). When a service disruption occurs, the first step is always to determine whether Microsoft is aware of the issue and what the expected resolution timeline is. The Microsoft 365 admin center provides two primary surfaces for this.

#### Microsoft 365 Service Health dashboard

The Service Health dashboard shows the real-time status of all Microsoft services in your tenant, including Windows 365 and its dependencies.

**Access**: Sign in to the [Microsoft 365 admin center](https://admin.microsoft.com) > **Health** > **Service health**

Key features:

- **Overview tab** – Shows all services, their current health state, and any active incidents or advisories
- **Issues for your organization to act on** – Issues detected in your environment that require administrator action
- **Active issues Microsoft is working on** – Incidents and advisories Microsoft is investigating or mitigating
- **Issue history** – Resolved incidents and advisories from the last 7 or 30 days, including Post-Incident Reports (PIR) with root cause analysis
- **Report an issue** – If your users are experiencing a problem not shown in Service Health, use this to notify Microsoft

Service incident status definitions:

| Status | Meaning |
|--------|---------|
| **Investigating** | Microsoft is aware of a potential issue and gathering information |
| **Service degradation** | Issue confirmed; service is slower or intermittently available |
| **Service interruption** | Significant issue affecting user access to the service |
| **Restoring service** | Root cause identified; corrective action in progress |
| **Extended recovery** | Fix in progress but will take time to reach all systems |
| **Service restored** | Corrective action complete; service is healthy |
| **Post-incident report published** | PIR available with root cause and preventive measures |

> [!NOTE]
> Service incident updates are provided on an hourly cadence. If a different cadence is required, it is stated in the Service Health posting.

#### Microsoft 365 Message Center

The Message Center communicates planned changes, new features, planned maintenance, and deprecations. It is distinct from Service Health (which covers unplanned incidents), but both are essential for Day 2 operations.

**Access**: [Microsoft 365 admin center](https://admin.microsoft.com) > **Health** > **Message center**

Use Message Center to:

- Track upcoming Windows 365 feature changes and deprecations
- Review planned maintenance that may affect Cloud PC availability
- Identify changes categorized as **Plan for change** that require administrator preparation
- Subscribe to a weekly digest email or major-update-only email notifications via **Preferences**

> [!TIP]
> Planned maintenance events are communicated through Message Center, not Service Health. Filter messages by the **Plan for change** category to identify actions required before a change takes effect.

#### Backup communication channels

When the Microsoft 365 admin center is inaccessible during a widespread outage, use these backup channels:

| Channel | URL / Handle | Purpose |
|---------|--------------|---------|
| **Service Health Status page** | [https://status.cloud.microsoft](https://status.cloud.microsoft) | Unauthenticated public status page; backup when admin center is unavailable |
| **@MSFT365Status on X (Twitter)** | [@MSFT365Status](https://x.com/MSFT365Status) | Real-time updates on active service events |
| **Azure Status** | [https://status.azure.com](https://status.azure.com) | Azure infrastructure status affecting Cloud PC compute and networking |
| **Microsoft 365 Admin mobile app** | Available on iOS and Android | View service health and maintenance status from mobile devices |

> [!IMPORTANT]
> The public status page at status.cloud.microsoft is only updated when an active service issue prevents customers from accessing the Microsoft 365 admin center. **Service Health** in the admin center is always the primary and most accurate source because it shows tenant-specific impact.

#### Recommended service health monitoring practices

| Practice | Detail |
|----------|--------|
| Assign Service Health review responsibility | Designate at least two administrators to review Service Health daily |
| Configure Message Center email preferences | Enable weekly digest or major-update-only emails for all Windows 365 administrators |
| Integrate with ITSM tooling | Use the [Microsoft 365 Service Communications API](https://learn.microsoft.com/graph/api/resources/service-communications-api-overview) to pull service health data into your monitoring or ITSM platform |
| Document an incident response plan | Define roles, communication templates, and escalation paths for when a Microsoft-declared outage affects your Cloud PCs |
| Review Post-Incident Reports | After major incidents, read the PIR in Service Health issue history to understand root cause and identify any tenant-side follow-up actions |
| Distinguish Microsoft vs. tenant issues | Before opening a support case, check Service Health to determine if the issue is already tracked; if it is, all updates will be provided there |

## Patch and update management

Keeping Cloud PCs patched is essential for security and compliance. Windows 365 Cloud PCs are managed through Intune update policies like physical Windows devices.

### Recommended update configuration

| Policy setting | Enterprise recommendation | Frontline recommendation |
|----------------|---------------------------|--------------------------|
| Microsoft product updates | Allow | Allow |
| Windows drivers | Allow | Allow |
| Quality update deferral period | 0–7 days | 9 days |
| Feature update deferral period | 0 days | 0 days |
| Automatic update behavior | Auto install at scheduled time | Auto install and restart at maintenance time |
| Active hours | Configured by IT | Configured by IT |
| Grace period | 2 days | 2 days |

### Maintenance windows

Maintenance windows let you schedule remote actions (restart, reprovision) to occur during a defined period, giving users advance notice.

**Configuration**: **Tenant administration** > **Cloud PC maintenance windows**

Key points:

- Users in-session receive notification about the impending maintenance period
- Maintenance windows apply to remote actions only; they don't control Windows Update or Intune payload delivery
- For urgent actions, bypass the maintenance window by not selecting **Use Cloud PC maintenance windows** when initiating the action

### Windows 365 Frontline update considerations

- Windows 365 Frontline Cloud PCs not used for 7 days are automatically powered on and synced with Windows Update policies
- The service cancels deallocation when a Windows Update is in progress, preventing interruption mid-patch
- Configure active hours carefully to avoid update reboots during shifts

### Update management best practices

1. Use [dynamic device groups](https://learn.microsoft.com/windows-365/enterprise/create-dynamic-device-group-all-cloudpcs) or [filters](https://learn.microsoft.com/windows-365/enterprise/create-filter) to target update policies specifically to Cloud PCs
2. Deploy update rings with staged rollout: pilot group → broad production
3. Monitor the resource performance report after feature updates to detect performance regressions
4. Coordinate with maintenance windows for required restart actions
5. Review Windows Update compliance reports in Intune to identify Cloud PCs missing patches

## Security operations

Maintaining security posture requires ongoing attention to baselines, conditional access, and threat detection.

### Windows 365 security baseline

Deploy the **Windows 365 Security Baseline version 24H1** from **Endpoint Security** > **Security Baselines** in the Intune admin center. Key settings include:

- Always prompt for password upon connection (Enabled)
- Require secure RPC communication (Enabled)
- Client connection encryption level (High)
- PowerShell Script Block Logging (Enabled)
- SmartScreen in Shell (Enabled)
- UAC policies (Enabled with Admin Approval Mode)

> [!TIP]
> Test the security baseline on a pilot group of Cloud PCs before broad deployment. Monitor for compatibility issues with line-of-business applications.

### Conditional Access policies

Target Conditional Access policies to the following app registrations:

| App | App ID | Purpose |
|-----|--------|---------|
| Windows 365 | `0af06dc6-e4b5-4f28-818e-e78e62d137a5` | User portal and Cloud PC actions (restart, restore) |
| Azure Virtual Desktop | `9cdead84-a844-4324-93f2-b2e6bb768d07` | Gateway authentication and session brokering |

Recommended policies:

1. **Require MFA** for all Cloud PC connections
2. **Require compliant device** for access from unmanaged endpoints
3. **Block access from untrusted locations** or countries where your organization doesn't operate
4. **Session controls**: Configure sign-in frequency appropriate for your risk tolerance

### Microsoft Defender for Endpoint

- Onboard Cloud PCs to Microsoft Defender for Endpoint
- Configure device compliance policies that set non-compliant state when threat level exceeds acceptable risk
- Combine with Conditional Access to block access to corporate resources from compromised Cloud PCs

### Ongoing security tasks

| Frequency | Task |
|-----------|------|
| Daily | Review security alerts from Defender for Endpoint |
| Weekly | Check compliance status of Cloud PC fleet in Intune |
| Monthly | Review Conditional Access sign-in logs for anomalies |
| Quarterly | Update security baselines to latest version; review and adjust CA policies |
| Annually | Conduct security posture review; assess new Windows 365 security features |

## Remote actions and remediation

Windows 365 supports a comprehensive set of remote actions for managing Cloud PCs at scale.

### Available remote actions

| Action | Bulk support | Use case |
|--------|:------------:|----------|
| Restart | Yes | Resolve performance issues, apply pending updates |
| Restore | Yes | Roll back to a known-good state after an incident |
| Resize | Yes | Address performance issues by scaling CPU/RAM |
| Reprovision | Yes | Full reset to resolve persistent issues |
| Collect diagnostics | Yes | Gather logs for troubleshooting |
| Place under review | Yes | Preserve Cloud PC state for investigation or compliance |
| Troubleshoot | Yes | Run connectivity health checks |
| Rename | No | Update Cloud PC display name |
| Create manual restore point | Yes | Backup before administrative changes |
| Share restore point to storage | Yes | Export to Azure Storage for archival |
| Power on/off (Frontline) | Yes | Manage Frontline Cloud PC availability |
| Sync | Yes | Force Intune policy sync |
| Update Windows Defender | No | Trigger signature update |

Bulk actions support up to 5,000 Cloud PCs per operation.

### Remediation workflow

```
Alert triggered → Triage severity → Identify affected Cloud PCs
     ↓
Connection issue? → Run Troubleshoot action → Review health check results
     ↓
Performance issue? → Check Resource Performance report → Resize if needed
     ↓
Persistent issue? → Create manual restore point → Restore to known-good state
     ↓
Unrecoverable? → Reprovision (destructive: user data lost)
```

### Proactive remediation with Intune

Use [Intune Remediations](https://learn.microsoft.com/mem/intune/fundamentals/remediations) (formerly Proactive Remediations) to detect and fix issues on Cloud PCs automatically:

- Detect stale credentials or expired certificates
- Monitor disk space and clean temporary files
- Verify required agents and services are running
- Check network connectivity to required endpoints

## Capacity and cost optimization

### Right-sizing Cloud PCs

Use the following data sources to determine optimal Cloud PC sizing:

1. **Resource performance report** – Identify Cloud PCs with consistent CPU/RAM spike times
2. **Cloud PC size recommendation report** – Review system-generated recommendations
3. **Utilization report** – Identify Cloud PCs with extremely low usage (candidates for Frontline or license removal)

### Cost optimization strategies

| Strategy | Action |
|----------|--------|
| **Identify unused Cloud PCs** | Review utilization report; reclaim licenses for Cloud PCs never or rarely connected |
| **Downsize over-provisioned Cloud PCs** | If resource performance shows consistently low utilization, resize to a smaller SKU |
| **Use Frontline licensing** | For shift workers or users not requiring 24/7 dedicated Cloud PCs |
| **Place under review with cold/archive tier** | When Cloud PCs are preserved for compliance, use lower-cost storage tiers |
| **Optimize DR coverage** | Assign Cross Region DR licenses only to users with documented RTO requirements |

### License management

- Monitor license consumption in the Microsoft 365 admin center
- Use the **Device Type** filter on the All Cloud PCs page to segment Enterprise, Frontline, Dev Box, and Power Automate Cloud PCs
- Configure concurrency buffers for Frontline to handle shift changes without over-licensing

## Operational automation

### Microsoft Graph API

Use the [Windows 365 Microsoft Graph APIs](https://learn.microsoft.com/windows-365/enterprise/permission-scopes) (v1.0 and Beta) to automate:

- Provisioning policy management
- Remote actions (restart, restore, resize)
- Reporting and data extraction
- Alert management

### Windows 365 Power Platform connector (preview)

The Windows 365 connector for Power Platform and Azure Logic Apps enables low-code automation of operational tasks:

- Automated license assignment workflows based on HR system events
- Scheduled report generation and distribution
- Automated remediation flows triggered by alert conditions
- Self-service Cloud PC management portals for end users

### Automation recommendations

| Scenario | Recommended approach |
|----------|---------------------|
| Alert-triggered remediation | Power Automate flow connected to email alerts |
| Scheduled reporting | Graph API with Azure Functions or Logic Apps |
| License lifecycle management | Power Automate with HR system connectors |
| DR readiness validation | Scheduled Graph API calls to check DR status report |
| Proactive device health | Intune Remediations scripts on schedule |

## Operational runbook template

Use this template to document procedures for your operations team:

### Incident: Cloud PCs not available

| Step | Action | Tool |
|------|--------|------|
| 1 | Verify alert in Intune admin center | Tenant administration > Cloud PC Alerts |
| 2 | Check scope: single user or multiple | Cloud PCs that aren't available report |
| 3 | Run **Troubleshoot** action on affected devices | Remote actions (bulk supported) |
| 4 | Review health check results | Cloud PC > Overview > Troubleshoot |
| 5 | If ANC-related: check Azure Network Connection health | Provisioning > Azure network connections |
| 6 | If persistent: create manual restore point, then Restore | Remote actions |
| 7 | If unrecoverable: Reprovision (coordinate with user) | Remote actions |
| 8 | Document root cause and resolution | ITSM tool |

### Incident: Regional outage

| Step | Action | Tool |
|------|--------|------|
| 1 | Check M365 Service Health for active incidents affecting Windows 365 or Azure dependencies | [Microsoft 365 admin center](https://admin.microsoft.com) > Health > Service health |
| 2 | If admin center is inaccessible, check backup status channels | [status.cloud.microsoft](https://status.cloud.microsoft), [status.azure.com](https://status.azure.com), [@MSFT365Status](https://x.com/MSFT365Status) |
| 3 | Confirm Microsoft-declared regional outage; note the incident ID and affected services | M365 Service Health incident details |
| 4 | Review Message Center for any related planned maintenance or advisory posts | M365 admin center > Health > Message center |
| 5 | Review DR readiness via status report | Cloud PCs cross region DR status report (Intune) |
| 6 | Activate cross region DR for affected users | Intune admin center > Cloud PC settings |
| 7 | Communicate to users: temporary Cloud PC available in ~4 hours; include incident ID for reference | Organizational communication channel |
| 8 | Monitor DR activation progress and Service Health updates (hourly cadence) | DR status report + M365 Service Health |
| 9 | After Microsoft declares "Service restored": Deactivate DR (or wait for 7-day auto-failback) | Intune admin center |
| 10 | Verify primary Cloud PCs are healthy post-failback | Cloud PC monitoring |
| 11 | Review Post-Incident Report (PIR) when published in Service Health issue history | M365 Service Health > Issue history |
| 12 | Conduct internal post-incident review; document lessons learned and any runbook updates | Operations team / ITSM tool |

### Incident: Performance degradation

| Step | Action | Tool |
|------|--------|------|
| 1 | Identify scope using connection health metrics | Cloud PC monitoring > Connection health |
| 2 | Check RTT and bandwidth for affected cohort | Connection quality report |
| 3 | Review CPU/RAM spike times | Resource performance report |
| 4 | If resource-constrained: initiate Resize | Remote actions |
| 5 | If network-related: evaluate client location and gateway | Connection quality report, Network config |
| 6 | If widespread: check for service incidents | [M365 admin center](https://admin.microsoft.com) > Health > Service health |

## Monitoring cadence summary

| Frequency | Activity |
|-----------|----------|
| **Real-time** | Alert notifications (console + email) |
| **Daily** | Review active alerts; check Cloud PC monitoring dashboard; review M365 Service Health and Message Center |
| **Weekly** | Review Cloud PCs not available report; check connection quality trends |
| **Monthly** | Review resource performance scores; assess utilization; validate DR readiness |
| **Quarterly** | Test DR activation/deactivation; review security baselines; adjust alert thresholds; update runbooks |
| **Annually** | Full operational review; evaluate new Windows 365 features; refresh BCDR documentation |

## Related content

- [Alerts in Windows 365](https://learn.microsoft.com/windows-365/enterprise/alerts)
- [Cloud PC monitoring overview](https://learn.microsoft.com/windows-365/enterprise/cloud-pc-monitoring-overview)
- [Cross region disaster recovery](https://learn.microsoft.com/windows-365/enterprise/cross-region-disaster-recovery)
- [Disaster Recovery Plus](https://learn.microsoft.com/windows-365/enterprise/disaster-recovery-plus)
- [Business continuity and disaster recovery](https://learn.microsoft.com/windows-365/business-continuity-disaster-recovery)
- [Point-in-time restore](https://learn.microsoft.com/windows-365/enterprise/restore-overview)
- [Windows 365 service resilience](https://learn.microsoft.com/windows-365/enterprise/resilience)
- [Remotely manage Cloud PCs](https://learn.microsoft.com/windows-365/enterprise/remotely-manage-cloud-pc)
- [Deploy security baselines](https://learn.microsoft.com/windows-365/enterprise/deploy-security-baselines)
- [Cloud PC maintenance windows](https://learn.microsoft.com/windows-365/enterprise/cloud-pc-maintenance-windows)
- [Set Conditional Access policies](https://learn.microsoft.com/windows-365/enterprise/set-conditional-access-policies)
- [Resource performance report](https://learn.microsoft.com/windows-365/enterprise/report-resource-performance)
- [Connection quality report](https://learn.microsoft.com/windows-365/enterprise/report-cloud-pc-connection-quality)
- [Security guidelines](https://learn.microsoft.com/windows-365/enterprise/security-guidelines)
- [How to check Microsoft 365 service health](https://learn.microsoft.com/microsoft-365/enterprise/view-service-health)
- [Message center in Microsoft 365](https://learn.microsoft.com/microsoft-365/admin/manage/message-center)
- [Service health and continuity](https://learn.microsoft.com/office365/servicedescriptions/office-365-platform-service-description/service-health-and-continuity)
- [Microsoft 365 Service Communications API](https://learn.microsoft.com/graph/api/resources/service-communications-api-overview)
