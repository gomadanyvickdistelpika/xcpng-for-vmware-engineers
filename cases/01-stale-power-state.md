# Case 1 – VM shows Running, but it is really off

**Ticket:** "Xen Orchestra shows my VM as Running. It is not, and I cannot start or stop it."
**VMware twin:** vCenter shows a VM in a wrong or invalid state → restart `hostd` and `vpxa` on the ESXi host.

## Why it happens
XAPI keeps a database of what it *believes* each VM is doing. If that database falls out of step with what Xen actually runs, XO shows the wrong state. The XCP-ng docs describe this exactly: "The XAPI database thinks that the VM is On / Off."

## Resolution
1. Compare belief and reality:
   ```bash
   xe vm-list params=name-label,power-state   # what XAPI believes
   xl list                                    # what Xen is really running
   ```
2. Read the toolstack log around the failed start/stop:
   ```bash
   grep <vm-uuid> /var/log/xensource.log | tail
   ```
3. Restart the toolstack. It restarts the management services only — **running VMs are not touched**:
   ```bash
   xe-toolstack-restart
   ```
4. Re-check with `xe vm-list`. Only if `xl list` proves the VM is off **on every host** and step 3 did not fix it:
   ```bash
   xe vm-reset-powerstate uuid=<vm-uuid> --force
   ```

## Do not
- Run `vm-reset-powerstate` before proving the VM is off. If it is still running somewhere, starting it again means two copies writing to one disk — severe corruption.

## Official documentation
- [XCP-ng – VM not in expected power state](https://docs.xcp-ng.org/troubleshooting/common-problems/#vm-not-in-expected-power-state)
- [XenServer 8 – Cope with machine failures (warning on `vm-reset-powerstate`)](https://docs.xenserver.com/en-us/xenserver/8/dr/machine-failures)
- [XCP-ng – Log files (xensource.log)](https://docs.xcp-ng.org/troubleshooting/log-files/)
