---
title: GPU Passthrough (KVM)
layout: default
parent: Homelab
nav_order: 3
---

# GPU Passthrough (KVM)

Passing an NVIDIA RTX 3080 through to a Windows 10 VM for gaming. The host runs Ubuntu with QEMU/KVM and libvirt.

## Hardware

- **CPU:** AMD (IOMMU support required — enable in BIOS as AMD-Vi)
- **GPU:** RTX 3080 LHR (passed to VM)
- **Host GPU:** AMD Raphael iGPU (used by host, keeps display working)

<!-- screenshot: Windows VM running a game / Sunshine stream -->
*![](img/gpu-passthrough-vm.png)*

## IOMMU Setup

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
- ASUS USB Bluetooth (0b05:190e) passed for wireless peripherals
- ASMedia USB controller (0d:00.0) passed for PS VR2 adapter
- Virtual VGA set to `primary="no"` so the RTX is the primary display

```bash
# bind GPU to vfio-pci before starting VM
echo "10de 2206" > /sys/bus/pci/drivers/vfio-pci/new_id
```

## Remote Access

Using **Sunshine** (on the Windows VM) + **Moonlight** (on client devices) for game streaming over LAN. Low latency, works well.

The RTX 3080 needs either a real monitor, a dummy HDMI plug, or the PS VR2 connected via DisplayPort — otherwise Sunshine can't capture the display.

VNC via `192.168.1.10:5900` as emergency fallback if something goes wrong with the GPU.

## PS VR2

Connected via DisplayPort to the RTX 3080 inside the VM. The USB adapter (ASMedia controller, 0d:00.0) is also passed through. Using **DeoVR** (free on Steam) for local VR video playback.

## Notes

- Driver installation in the VM is straightforward — Windows just picks up the RTX 3080 normally
- Biggest gotcha: IOMMU group isolation. If your GPU shares a group with something else, passthrough won't work cleanly
- The VM definition lives in `virsh` — back it up with `virsh dumpxml win10-2025-11-1 > win10.xml`
