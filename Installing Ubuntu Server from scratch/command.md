```
sudo apt update && sudo apt upgrade
```

```
sudo apt install cifs-utils nfs-common qemu-guest-agent
```

```
sudo apt install ca-certificates curl gnupg lsb-release ntp htop zip unzip gnupg apt-transport-https net-tools ncdu apache2-utils btop mc alsa-utils pciutils
```

```
sudo timedatectl set-timezone Europe/Moscow
```

```
sudo nano /etc/sysctl.conf

vm.swappiness=10
vm.vfs_cache_pressure = 50
fs.inotify.max_user_watches=524288
vm.overcommit_memory = 1

```

install docker engine
