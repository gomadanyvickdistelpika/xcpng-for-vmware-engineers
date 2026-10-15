# XCP-ng for VMware Engineers

A small, hands-on lesson for vSphere engineers moving to the **Vates stack** (XCP-ng + Xen Orchestra).

You install a simulated XCP-ng 8.3 host, log in over SSH, then resolve **six common support cases** using the same commands you would type on a real host. Every case is mapped to its VMware twin and linked to the official Vates / XCP-ng / Xen Orchestra documentation.

**▶ Open the lab:** https://gomadanyvickdistelpika.github.io/xcpng-for-vmware-engineers/
(or open `docs/index.html` in any browser — it is one self-contained file, no install)

> **Honest limits.** The lab is a simulation built from public documentation. Screens are reconstructions and log lines are illustrative. Before running a command in production, confirm it with `xe help <command>` on your version. Not affiliated with or endorsed by Vates.

---

## Who this is for

- You know ESXi, vCenter, datastores, HA and `vm-support`.
- You are new to XCP-ng and want to know *where things live* and *what to type* when a ticket lands.
- Time needed: about 60–90 minutes.

## Lesson plan

| Step | What you do | Where |
|---|---|---|
| 1 | Learn the vocabulary map (below) | This page |
| 2 | Install a host and set a static management IP | Lab tab 1 – *Install* |
| 3 | Use xsconsole, SSH and `xe` basics | Lab tab 2 – *Host console* |
| 4 | Create a VM in XO Lite | Lab tab 3 – *XO Lite* |
| 5 | Resolve six tickets in a simulated shell | Lab tab 4 – *Six cases* |
| 6 | Review resolutions and official docs | Lab tab 5 + [`cases/`](cases/) |

## 1. The words you already know

| VMware | XCP-ng / Xen Orchestra |
|---|---|
| ESXi host | XCP-ng host (Xen hypervisor + dom0 control domain) |
| vCenter | Xen Orchestra (XOA) |
| Host Client | XO Lite (built into the host) |
| hostd / vpxa | XAPI toolstack (`xe-toolstack-restart`) |
| Cluster | Pool, run by the pool master (now called *pool coordinator*) |
| Datastore | SR — storage repository |
| Datastore mounted on a host | PBD — plugged / unplugged |
| VMDK | VDI (VHD or qcow2) |
| vmnic / vNIC / port group | PIF / VIF / network (bridge, e.g. `xenbr0`) |
| VMware Tools | Xen guest tools / PV drivers |
| vMotion / Storage vMotion | Live migration / storage migration |
| Snapshot consolidation | Coalesce |
| esxcli, PowerCLI | `xe` CLI, `xo-cli` |
| vm-support | `xen-bugtool` (status report) |

## 2. Where the logs are

| VMware | XCP-ng | Tells you |
|---|---|---|
| hostd.log | `/var/log/xensource.log` | XAPI, the host agent |
| vmkernel.log | `/var/log/kern.log`, `xl dmesg` | Kernel, hardware, Xen |
| vmkernel.log (storage) | `/var/log/SMlog` | Storage manager, coalesce |
| — | `/var/log/daemon.log` | xenopsd, migration, storage motion |
| fdm.log | `/var/log/xha.log` | HA, heartbeat, fencing |
| vpxd.log | Xen Orchestra logs | Central management |

Source: [XCP-ng docs – Log files](https://docs.xcp-ng.org/troubleshooting/log-files/)

## 3. Six common cases

| # | Symptom | VMware twin | First log | Key command | Write-up |
|---|---|---|---|---|---|
| 1 | VM shows Running but is off | Invalid VM state → restart hostd/vpxa | xensource.log | `xe-toolstack-restart` | [01](cases/01-stale-power-state.md) |
| 2 | Pool master lost | HA elects a new primary | xha.log | `xe pool-emergency-transition-to-master` | [02](cases/02-pool-master-lost.md) |
| 3 | Backup skipped: unhealthy VDI chain | Disk consolidation needed | SMlog | coalesce, SR free space | [03](cases/03-backup-skipped-coalesce.md) |
| 4 | SR disconnected | Datastore inaccessible / APD | SMlog | `xe pbd-plug` | [04](cases/04-sr-disconnected.md) |
| 5 | Host crashed and rebooted | PSOD, vm-support | kern.log, `xl dmesg` | `xen-bugtool --yestoall` | [05](cases/05-host-crash.md) |
| 6 | `HOST_NOT_ENOUGH_FREE_MEMORY` | Insufficient resources | `xl info` | `xe vm-migrate` / `vm-memory-limits-set` | [06](cases/06-not-enough-memory.md) |

Every write-up follows the same shape: **symptom → why it happens → resolution steps → what not to do → official documentation**.

## 4. Moving VMs off VMware

- [Vates KB VKB-1128 – Migration from VMware to XCP-ng](https://docs.vates.tech/kb/vkb-1128/)
- [Xen Orchestra – VMware migration (V2V) guide](https://docs.xen-orchestra.com/guides/v2v-migration-guide)
- [XCP-ng docs – Migrate to XCP-ng](https://docs.xcp-ng.org/installation/migrate-to-xcp-ng)

## Official references

- [XCP-ng documentation](https://docs.xcp-ng.org/) · [Troubleshooting](https://docs.xcp-ng.org/troubleshooting/)
- [Xen Orchestra documentation](https://docs.xen-orchestra.com/)
- [Vates Knowledge Base](https://docs.vates.tech/kb/)
- [XenServer 8 docs](https://docs.xenserver.com/en-us/xenserver/8/) — same XAPI toolstack; used where XCP-ng docs defer to it

## Repository layout

```
docs/index.html   the interactive lab (served by GitHub Pages)
cases/            one write-up per case, with official doc links
```

## Feedback

Corrections from people who run XCP-ng in production are very welcome — open an issue or a pull request.

## Author

Goma Dany Vick Distel Pika — Senior Technical Support Engineer (VMware vSphere / VCF background), learning the Vates stack in public.

## Licence

MIT — see [LICENSE](LICENSE). XCP-ng, Xen Orchestra and Vates are trademarks of their owners.
