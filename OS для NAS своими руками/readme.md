 
Samba (SMB support): 
```
apt install samba -y
```
NFS Server (NFS support):
```
apt install nfs-kernel-server -y
```

Virtualization tools (KVM/QEMU):
```
apt install qemu-kvm virt-manager libvirt-clients bridge-utils libvirt-daemon-system virtinst -y
```
Containerization (Podman):
```
apt install podman -y
```
ZFS Utilities:
```
apt install zfsutils-linux -y
```
Hardware Monitoring:
```
apt install lm-sensors -y
sensors-detect
```
```
apt install cockpit -y
systemctl enable --now cockpit.socket
```
```
apt install cockpit-machines -y
```
```
apt install cockpit-podman -y
```
```
apt install cockpit-sosreport -y
```

```
git clone https://github.com/45drives/cockpit-zfs-manager.git
sudo cp -r cockpit-zfs-manager/zfs /usr/share/cockpit
```
```
curl -sSL https://repo.45drives.com/setup | sudo bash
apt update
apt install cockpit-file-sharing -y
```
```
apt install cockpit-identities -y
```
```
apt install cockpit-navigator -y
```
```
apt install cockpit-podman -y
```
```
wget https://github.com/ocristopfer/cockpit-sensors/releases/latest/download/cockpit-sensors.tar.xz && \
tar -xf cockpit-sensors.tar.xz cockpit-sensors/dist && \
mv cockpit-sensors/dist /usr/share/cockpit/sensors && \
rm -r cockpit-sensors && \
rm cockpit-sensors.tar.xz
```
