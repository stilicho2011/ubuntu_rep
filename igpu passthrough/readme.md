
Install the required drivers on the Proxmox host.

```
apt install -y intel-opencl-icd
```

Edit grub

Code:
```
nano /etc/default/grub
```

Find the line that starts with GRUB_CMDLINE_LINUX_DEFAULT and change to following:
Code:

```GRUB_CMDLINE_LINUX_DEFAULT="quiet intel_iommu=on i915.enable_gvt=1"```

Execute command:
Code:

```update-grub```


reboot host

Validate changes. It showed IOMMU enabled
Code:
```dmesg | grep -e DMAR -e IOMMU```


Edit etc modules
Code:
```
nano /etc/modules
```


Code:

`#Modules required for PCI passthrough
    vfio
    vfio_iommu_type1
    vfio_pci
#    vfio_virqfd`

`# Modules required for Intel GVT
    kvmgt
    exngt
    Vfio-mdev`

Execute command:
Code:
```
update-initramfs -u -k all
```
Reboot

Your VGA card should be visible with command
Code:
```
lspci -nnv | grep VGA
```

Enable GUC
Code:
```
echo "options i915 enable_guc=3" >> /etc/modprobe.d/i915.conf
```

Create LXC container as plivilaged and add parameters to configuration
Code:

```
nano /etc/pve/lxc/<container number>.conf
```

Code:
```
lxc.cgroup2.devices.allow: c 226:0 rwm
lxc.cgroup2.devices.allow: c 226:128 rwm
lxc.mount.entry: /dev/dri/renderD128 dev/dri/renderD128 none bind,optional,create=file
```


On LXC container


Update and upgrade system
Code:
```
apt update && apt upgrade -y
```
