---
title: Homelab - GPU Passthrough (KVM)
layout: default
nav_order: 5
---

# GPU Passthrough (KVM)
So i wanted to game, but i really don't like microslop windows and avoid using it, but at the moment VR gaming is not yet there. Also the point is using my gaming PC as server, and my thinkpad may run balatro and not much more. So i figured:
passing an NVIDIA RTX 3080 through to a Windows 10 VM for gaming is the best choice. The host runs Ubuntu with QEMU/KVM and libvirt.

## Hardware

- **CPU:** AMD (IOMMU support required — enable in BIOS as AMD-Vi)
- **GPU:** RTX 3080 LHR (passed to VM)
- **Host GPU:** Intel something-iris

<!-- screenshot: Windows VM running a game / Sunshine stream -->
*![](img/gpu-passthrough-vm.png)*

## IOMMU Setup
(This is mostly done with claude and some googling)

Enable IOMMU in BIOS, then add to kernel parameters:

```
GRUB_CMDLINE_LINUX_DEFAULT="amd_iommu=on iommu=pt"
```

Check IOMMU groups to make sure the GPU and its audio device are isolated (same group is fine, other devices in the same group are a problem):

```bash
for d in /sys/kernel/iommu_groups/*/devices/*; do
  echo "Group $(cut -d/ -f5 <<< $d): $(lspci -nns ${d##*/})"
done | sort -V
```

## Passthrough Config

The VM is managed via `virsh`. Key parts of the XML config:

- GPU (01:00.0) and its HDMI audio (01:00.1) passed via `<hostdev>` with `<driver name='vfio'/>`
- ASUS USB Bluetooth passed for wireless peripherals
- USB passthrough for PS VR2 adapter
- Virtual VGA set to `primary="no"` so the RTX is the primary display

```bash
# bind GPU to vfio-pci before starting VM
echo "10de 2206" > /sys/bus/pci/drivers/vfio-pci/new_id
```

## Remote Access

Using **Sunshine** (on the Windows VM) + **Moonlight** (on client devices) for game streaming over LAN. Low latency, works well - most of the times when connected via ethernet, on wifi not so much, but i haven't looked in why so, idea is it should run at least 1080p with ok bitrate.

The RTX 3080 dummy HDMI or DP plug, i use some old hdmi->vga adapter. 

If something goes bad with gpu i can use cockpit or cli and just reboot.

## Notes

- Driver installation in the VM is straightforward — Windows just picks up the RTX 3080 normally
- Biggest gotcha: IOMMU group isolation. If your GPU shares a group with something else, passthrough won't work cleanly
- The VM definition lives in `virsh` — back it up with `virsh dumpxml win10-2025-11-1 > win10.xml`
