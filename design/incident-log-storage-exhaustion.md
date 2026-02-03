# Incident Report — Log Storage Exhaustion

## Summary

During controlled stress testing of the centralized logging server (`log01`), the root filesystem reached capacity due to unbounded log growth.

The system remained operational; however, the event exposed a critical architectural risk: logging workloads shared a failure domain with the operating system.

This condition had the potential to degrade system stability, disrupt telemetry collection, and complicate recovery operations.

The incident prompted a live storage re-architecture to isolate `/var/log` onto a dedicated logical volume.

---

## Environment

- Platform: RHEL 9  
- Logging Transport: rsyslog over TCP (514)  
- SELinux: Enforcing  
- Firewall: firewalld (allow-list model)  
- Architecture: Bastion-mediated internal network  

---

## Trigger

Disk pressure was intentionally generated to observe system behavior under constrained storage conditions.

A large file was written to `/var/log`, causing the root filesystem to approach full utilization.

---

## Observed Behavior

As disk utilization increased:

- Logging throughput began to degrade  
- Write operations slowed  
- Risk to audit continuity increased  

Despite the storage pressure:

- SSH access remained available  
- Administrative control paths were unaffected  
- The kernel remained responsive  

This confirmed that the failure was progressive rather than abrupt — a common characteristic of filesystem exhaustion events.

---

## Root Cause

`/var/log` resided on the root logical volume.

Logs represent an unpredictable growth workload and should not compete with operating system resources.

The architecture allowed log expansion to threaten core system functionality.

---

## Impact Assessment

**Immediate Risk:**
- Potential loss of telemetry  
- Delayed security visibility  
- Reduced operational awareness  

**Escalation Risk:**
- Service instability  
- Administrative friction during recovery  
- Increased mean time to resolution  

While no outage occurred, the architecture violated a fundamental resilience principle: isolate volatile storage consumers from critical system partitions.

---

## Engineering Response

The system was re-architected live without rebuilding the server.

Actions executed:

- Added a new virtual disk to `log01`  
- Extended the existing volume group  
- Created a dedicated logical volume for `/var/log`  
- Formatted the volume using XFS  
- Migrated log data with `rsync`  
- Restored SELinux contexts  
- Updated `/etc/fstab` for persistent mounting  
- Reboot-tested the system  
- Validated rsyslog functionality  

The migration preserved system continuity while removing the shared failure domain.

---

## Architectural Correction

**Before:**

```
LV Root  -> /
           └── /var/log
```

**After:**

```
LV Root   -> /
LV VarLog -> /var/log
```

This separation ensures that log growth cannot impair operating system stability.

---

## Validation

A second disk exhaustion test targeted the new `/var/log` volume.

### Result

- Root filesystem remained healthy  
- OS responsiveness was unchanged  
- SSH access persisted  
- Logging pipeline continued functioning  

The system demonstrated graceful degradation rather than systemic failure.

Failure-domain isolation was successfully validated.

---

## Lessons Learned

### 1. Logs Are High-Entropy Storage Consumers
Growth patterns are unpredictable and must be architecturally contained.

### 2. Failure Should Be Directional
Well-designed infrastructure degrades in isolated layers rather than collapsing globally.

### 3. Live Migration Is Preferable to Rebuild
When executed carefully, LVM enables architectural correction without operational disruption.

### 4. SELinux Must Be Preserved During Storage Changes
Security posture should never regress as a side effect of infrastructure work.

---

## Preventative Measures

- Dedicated logical volume for `/var/log`  
- Continued monitoring of filesystem utilization  
- Planned log rotation tuning  
- Capacity awareness during future expansions  

---

## Strategic Outcome

This event materially improved the resilience of the logging platform.

The environment now reflects a production-aligned storage pattern where volatile workloads are isolated from core operating system resources.

The incident reinforced a guiding infrastructure principle:

> Systems should fail predictably, recover quickly, and preserve administrative control paths.
