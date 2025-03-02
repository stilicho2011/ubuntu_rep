Edit the sources list

```
nano /etc/apt/sources.list
```

Add these lines at the bottom:

```
# Proxmox VE pve-no-subscription repository provided by proxmox.com
# NOT recommended for production use
deb http://download.proxmox.com/debian/pve bookworm pve-no-subscription

# Security updates
deb http://security.debian.org/debian-security bookworm-security main contrib
```

Disable the Production Repository

Next, disable the production repository by commenting out these lines in pve-enterprise.list:

```
nano /etc/apt/sources.list.d/pve-enterprise.list
```

Comment out this line:

```
#deb https://enterprise.proxmox.com/debian/pve bookworm pve-enterprise
```

Configure Ceph for No-Subscription

Open the Ceph repository file:

```
nano /etc/apt/sources.list.d/ceph.list
```

Comment out the enterprise repository: by adding this # in front of the next line

```
#deb https://enterprise.proxmox.com/debian/ceph-quincy bookworm enterprise
```

Add the no-subscription repository:

```
deb http://download.proxmox.com/debian/ceph-quincy bookworm no-subscription
```

```
apt-get update && apt-get upgrade -y
```


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

Disable No-Subscription Pop-Up
```
sed -Ezi.bak "s/(Ext.Msg.show\(\{\s+title: gettext\('No valid sub)/void\(\{ \/\/\1/g" /usr/share/javascript/proxmox-widget-toolkit/proxmoxlib.js
```
