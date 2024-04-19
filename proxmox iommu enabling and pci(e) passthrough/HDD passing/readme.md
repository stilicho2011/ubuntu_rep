https://pve.proxmox.com/pve-docs/chapter-qm.html#_general_requirements

How to determing if you are using grub or cmdline
use `efibootmgr -v`

If you see something like “File(\EFI\SYSTEMD\SYSTEMD-BOOTX64.EFI)” then you are using systemd, not GRUB.

If you have GRUB edit config file:

```
nano /etc/default/grub
```

For Intel CPUs Intel CPUs add quiet intel_iommu=on: 
```
GRUB_CMDLINE_LINUX_DEFAULT="quiet intel_iommu=on pt=on"
```

For AMD CPUs add quiet amd_iommu=on:

```
GRUB_CMDLINE_LINUX_DEFAULT="quiet amd_iommu=on pt=on"
```

Then update GRUB
```
update-grub
```

If you have cmdline edit config file 
```
nano /etc/kernel/cmdline
```

insert in the end of the line
`quiet amd_iommu=on iommu=pt`

Then refresh boot tool
```
proxmox-boot-tool refresh
```
Then reboot


Add Modules

```
nano /etc/modules
```

Insert

```
vfio
vfio_iommu_type1
vfio_pci
vfio_virqfd #not necessary if kernel 6.2
```
Update modules
```
update-initramfs -u -k all
```
Reboot

Verify

`dmesg | grep -e DMAR -e IOMMU` or
`dmesg | grep -e DMAR -e IOMMU -e AMD-Vi`

GPU Isolation From the Host (amend the below to include the IDs of the device you want to isolate)

Get device IDs: ` lspci -nn `

```
echo "options vfio-pci ids=10de:____,10de:____ disable_vga=1" > /etc/modprobe.d/vfio.conf
```

Blacklist GPU drivers 

```
echo "blacklist radeon" >> /etc/modprobe.d/blacklist.conf 
echo "blacklist nouveau" >> /etc/modprobe.d/blacklist.conf 
echo "blacklist nvidia" >> /etc/modprobe.d/blacklist.conf 
echo "blacklist nvidiafb" >> /etc/modprobe.d/blacklist.conf
echo "blacklist nvidia_drm" >> /etc/modprobe.d/blacklist.conf
echo "blacklist i915" >> /etc/modprobe.d/blacklist.conf
```
Reboot 

Passing HDD

```
ls -n /dev/disk/by-id/
```
then
```
/sbin/qm set [VM-ID] -virtio2 /dev/disk/by-id/[DISK-ID]
```



