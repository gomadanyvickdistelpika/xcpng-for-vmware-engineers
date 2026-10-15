# Case 5 – Host crashed and rebooted by itself

**Ticket:** "A host rebooted by itself overnight and we do not know why."
**VMware twin:** a PSOD — read the purple screen and core dump, collect a `vm-support` bundle.

## Why it happens
Four different causes look the same from outside:
1. **Hardware** fault (machine check / MCE, memory, CPU, PSU)
2. **Xen hypervisor** crash
3. **dom0 kernel** panic
4. **HA fencing** — the host restarted itself after losing its heartbeat

## Resolution
1. When did it happen?
   ```bash
   last -x | head
   ```
2. Hardware and hypervisor messages (Xen has its own log, separate from dom0):
   ```bash
   xl dmesg | grep -i -E "mce|error|panic"
   grep -i -E "mce|panic" /var/log/kern.log
   ```
   Also check the BMC (iLO / iDRAC / IPMI) event log.
3. Rule HA fencing in or out:
   ```bash
   grep -i -E "fence|watchdog" /var/log/xha.log
   ```
4. Crash dumps are saved in `/var/crash` at the next boot:
   ```bash
   xe host-crashdump-list
   ```
   They sit on the boot disk — delete them (`xe host-crashdump-destroy uuid=<uuid>`) after analysis.
5. Collect the status report **before** any further reboot and share it privately with support:
   ```bash
   xen-bugtool --yestoall
   ```

## Do not
- Reboot again before the evidence is collected.
- Post the status report publicly — it can contain sensitive data.

## Official documentation
- [XCP-ng – Host crashes and crash dumps](https://docs.xcp-ng.org/troubleshooting/advanced/#host-crashes-and-crash-dumps)
- [XCP-ng – Reading the hypervisor state (`xl dmesg`)](https://docs.xcp-ng.org/troubleshooting/advanced/#reading-the-hypervisor-state)
- [XCP-ng – Produce a status report (`xen-bugtool`)](https://docs.xcp-ng.org/troubleshooting/log-files/#produce-a-status-report)
- [XCP-ng – HA: my host rebooted, why?](https://docs.xcp-ng.org/troubleshooting/troubleshooting-ha/#my-host-rebooted-why-did-it-reboot)
