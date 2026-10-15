# Case 6 – VM will not start: HOST_NOT_ENOUGH_FREE_MEMORY

**Ticket:** "Starting the VM fails with HOST_NOT_ENOUGH_FREE_MEMORY."
**VMware twin:** "Insufficient resources" / power-on blocked by memory admission control.

## Why it happens
The host has less free memory than the VM asks for. Running VMs (and dom0) hold their memory, so *free* is well below *total*. The two numbers in the error are **bytes needed** and **bytes free**.

## Resolution
1. Read the error and convert the bytes to GiB.
2. Check the host:
   ```bash
   xe host-list params=name-label,memory-total,memory-free
   xl info | grep -E "total_memory|free_memory"
   ```
3. Free memory safely — live-migrate another VM away, or start this VM on a host with room:
   ```bash
   xe vm-migrate vm=<other-vm> host=<other-host>
   ```
4. Or reduce this VM (the four limits must stay in order static-min ≤ dynamic-min ≤ dynamic-max ≤ static-max):
   ```bash
   xe vm-memory-limits-set uuid=<vm-uuid> static-min=2GiB dynamic-min=2GiB dynamic-max=4GiB static-max=4GiB
   ```
   Keep `static-min` at its current value (`xe vm-param-get uuid=<vm-uuid> param-name=memory-static-min`) — it is set per guest OS and XenServer docs advise not changing it.
5. Longer term: size the pool so one host can fail and its VMs still fit (N+1).

## Do not
- Shut down a production VM to make room without the owner's approval.
- Try to force the start past the check.

## Official documentation
- [XenServer 8 – VM memory (static and dynamic limits)](https://docs.xenserver.com/en-us/xenserver/8/vms/vm-memory)
- [XCP-ng – xe CLI (`xe help`)](https://docs.xcp-ng.org/management/manage-locally/cli/)
