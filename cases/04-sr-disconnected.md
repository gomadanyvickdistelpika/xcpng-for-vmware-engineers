# Case 4 – SR disconnected, VMs will not start

**Ticket:** "Our shared storage shows as disconnected and the VMs on it will not start."
**VMware twin:** Datastore inaccessible / All Paths Down — you read `vmkernel.log` and run `esxcli storage core path list`.

## Why it happens
An SR (the datastore) is attached to each host through a **PBD**. If the PBD is unplugged, that host cannot see the SR. The usual root cause is outside XCP-ng: NFS server down, export permissions, iSCSI target/CHAP, VLAN or MTU mismatch.

## Resolution
1. Find the detached PBD:
   ```bash
   xe pbd-list sr-uuid=<sr-uuid> params=uuid,host-name-label,currently-attached
   ```
2. Read the storage manager log for the mount/login error:
   ```bash
   tail -50 /var/log/SMlog
   ```
3. Prove reachability **from the host**: `ping <storage-ip>`, `showmount -e <nfs-server>` for NFS, or iSCSI discovery for block storage. Check MTU end to end.
4. Fix the cause with the storage / network team, then re-plug:
   ```bash
   xe pbd-plug uuid=<pbd-uuid>
   ```
5. Changing the SR connection settings (server, path, target) means destroying and recreating the PBD — **with the VMs on that SR stopped**.

## Do not
- Unplug the PBD of an SR with running VMs.
- Retry `pbd-plug` in a loop before the cause is fixed — it will fail the same way.

## Official documentation
- [XCP-ng – Storage: NFS](https://docs.xcp-ng.org/storage/#nfs)
- [XCP-ng – How to modify an existing SR connection (PBD)](https://docs.xcp-ng.org/storage/#how-to-modify-an-existing-sr-connection)
- [XCP-ng – iSCSI troubleshooting](https://docs.xcp-ng.org/troubleshooting/storage/iscsi-troubleshooting/)
- [Vates KB VKB-1147 – Diagnose and fix an MTU mismatch](https://docs.vates.tech/kb/vkb-1147)
