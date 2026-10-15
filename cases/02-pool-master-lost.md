# Case 2 – Pool master (coordinator) lost

**Ticket:** "Our pool master crashed. The VMs on the other hosts still run, but XO cannot manage the pool."
**VMware twin:** vSphere HA elects a new primary host; if vCenter is down you fall back to each host's Host Client.

## Why it happens
One host — the pool master, now called the *pool coordinator* — holds the pool database. Members keep running their VMs but cannot be managed without it.
- **HA enabled:** another coordinator is elected automatically.
- **HA disabled:** members wait for the coordinator to come back. If it is truly dead, you promote a member by hand.

## Resolution
1. Read `/var/log/xha.log` on a member (heartbeat, fencing). If HA already elected a new coordinator, stop here and verify.
2. **Prove the old master is dead**: ping it, check its BMC (iLO / iDRAC / IPMI) power state. Promoting while it is alive gives you two masters.
3. On one surviving member:
   ```bash
   xe pool-emergency-transition-to-master
   ```
4. Point the other members at the new coordinator — either:
   ```bash
   xe pool-recover-slaves                                        # on the new coordinator (XenServer docs)
   xe pool-emergency-reset-master master-address=<new-master>    # on each member (XCP-ng docs)
   ```
5. Check the pool's default SR is valid (`xe pool-param-list`), restart VMs that ran on the dead host (see Case 1 warning), then repair the old host and re-join it.
6. If HA itself is the problem: `xe host-emergency-ha-disable force=true` then `xe-toolstack-restart`; re-enable HA only after the root cause is fixed.

## Do not
- Promote a member before proving the old master is down.
- Re-enable HA before you know why it misbehaved.

## Official documentation
- [XCP-ng – HA troubleshooting: changing pool coordinators](https://docs.xcp-ng.org/troubleshooting/troubleshooting-ha/#changing-pool-coordinators)
- [XCP-ng – HA troubleshooting: disabling HA](https://docs.xcp-ng.org/troubleshooting/troubleshooting-ha/#disabling-ha)
- [XenServer 8 – Pool coordinator failures](https://docs.xenserver.com/en-us/xenserver/8/dr/machine-failures)
- [Vates KB VKB-1096 – Add a new slave to a running pool and match the patch level](https://docs.vates.tech/kb/vkb-1096)
