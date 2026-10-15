# Case 3 – Backup skipped: "unhealthy VDI chain"

**Ticket:** "Our backup job says skipped: the VDI chain is too long or unhealthy."
**VMware twin:** "Virtual machine disks consolidation is needed" after snapshot-based backups.

## Why it happens
Every backup takes a snapshot. When a snapshot is deleted, XCP-ng must **coalesce** (merge) the VDI chain. If snapshots arrive faster than merges finish, the chain grows toward the fixed limit of **30**. Xen Orchestra checks the chain before snapshotting and skips the run on purpose — it is a protection for the SR, not a failure.

## Resolution
1. Look for coalesce errors (include rotated logs):
   ```bash
   grep -i -E "exception|error|coales" /var/log/SMlog*
   ```
2. Check SR free space — coalesce needs room. The XO docs suggest keeping 30% or more free, depending on VM size.
3. Check whether a coalesce is running:
   ```bash
   ps axf | grep vhd
   ```
4. In XO, **Rescan all disks** on the SR and watch the *disks to coalesce* list (SR → Advanced) shrink. The next backup run normally succeeds.
5. Reduce snapshot frequency / retention so merges can keep up.
6. Last resort: migrate the VM's disks to another SR — this forces a coalesce.

## Do not
- Delete VDIs in the chain by hand.
- Bypass the protection to "make the backup green".

## Official documentation
- [Xen Orchestra – Backup troubleshooting (VDI chain protection)](https://docs.xen-orchestra.com/backups-and-dr/backup_troubleshooting)
- [XCP-ng – Storage: Coalesce](https://docs.xcp-ng.org/storage/#coalesce)
- [XCP-ng – Log files (SMlog)](https://docs.xcp-ng.org/troubleshooting/log-files/)
