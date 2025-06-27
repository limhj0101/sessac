vagrant, github, vscode

git연습

https://learngitbranching.js.org/?locale=ko

수업 강사님 드라이브

https://drive.google.com/drive/folders/1ypIAcyVIDbFt-tgSKGHhIDFyrNw4nqyz



`print`

```bash
#!/bin/bash
echo "hello"
```

vagrantfile (1)
```
# -*- mode: ruby -*-
# vi: set ft=ruby :
vm_image = "nobreak-labs/rocky-9"
vm_subnet = "192.168.56."

Vagrant.configure("2") do |config|
  config.vm.synced_folder ".", "/vagrant", disabled: true

  config.vm.define "sessac1" do |node|
    node.vm.box = vm_image
    node.vm.provider "virtualbox" do |vb|
      vb.name = "sessac1"
      vb.cpus = 2
      vb.memory = 4096
    end

    node.vm.network "private_network", ip: vm_subnet + "11", nic_type: "virtio"
    node.vm.hostname = "sessac1"
  end
end
```

vagrantfile(2)
```
# -*- mode: ruby -*-
# vi: set ft=ruby :
#vm_image = "nobreak-labs/rocky-9"
vm_image = "nobreak-labs/rocky-9"
vm_subnet = "192.168.56."

install_tools_script = <<-SCRIPT
# lsd
LSD_VERSION="1.1.5"
curl -LO https://github.com/lsd-rs/lsd/releases/download/v${LSD_VERSION}/lsd-v${LSD_VERSION}-x86_64-unknown-linux-gnu.tar.gz
tar -xzf lsd-v${LSD_VERSION}-x86_64-unknown-linux-gnu.tar.gz
mv lsd-v${LSD_VERSION}-x86_64-unknown-linux-gnu/lsd /usr/local/bin/
rm -rf lsd-v${LSD_VERSION}-x86_64-unknown-linux-gnu.tar.gz lsd-v${LSD_VERSION}-x86_64-unknown-linux-gnu

# bat
BAT_VERSION="0.24.0"
curl -LO https://github.com/sharkdp/bat/releases/download/v${BAT_VERSION}/bat-v${BAT_VERSION}-x86_64-unknown-linux-gnu.tar.gz
tar -xzf bat-v${BAT_VERSION}-x86_64-unknown-linux-gnu.tar.gz
mv bat-v${BAT_VERSION}-x86_64-unknown-linux-gnu/bat /usr/local/bin/
rm -rf bat-v${BAT_VERSION}-x86_64-unknown-linux-gnu.tar.gz bat-v${BAT_VERSION}-x86_64-unknown-linux-gnu

# Set aliases
ALIASES="
alias ls='lsd'
alias ll='lsd -l'
alias la='lsd -a'
alias lla='lsd -la'
alias cat='bat -pp'
"

# Add aliases to vagrant
echo "$ALIASES" >> /home/vagrant/.bashrc
chown vagrant:vagrant /home/vagrant/.bashrc

# Add aliases to root
echo "$ALIASES" >> /root/.bashrc

# Add aliases to skel for new users
echo "$ALIASES" >> /etc/skel/.bashrc

# Apply bashrc
source /root/.bashrc
sudo -u vagrant bash -c 'source /home/vagrant/.bashrc'
SCRIPT

Vagrant.configure("2") do |config|
  config.vm.synced_folder ".", "/vagrant", disabled: true

  config.vm.define "sessac1" do |node|
    node.vm.box = vm_image
    node.vm.provider "virtualbox" do |vb|
      vb.name = "sessac1"
      vb.cpus = 2
      vb.memory = 2048
    end

    node.vm.network "private_network", ip: vm_subnet + "11", nic_type: "virtio"
    node.vm.hostname = "sessac1"
    node.vm.provision "shell", inline: install_tools_script, name: "install_tools"
  end
end
```

# 1장
## 사용자 생성
```
    1  cat /etc/passwd
    2  cat /etc/shadow
    3  sudo cat /etc/shadow
    4  cat /etc/group
    5  cat /etc/gshadow
    6  sudo cat /etc/gshadow
    7  ls -l vagrant
    8  ls -l
    9  ls -l /etc
   10  ls man
   11  man ls
   12  alias ll
   13  useradd user01
   14  su - useradd user01
   15  su -
   16  sudo useradd user01
   17  tail -1 /etc/passwd
   18  tail -1 /etc/shadow
   19  sudo tail -1 /etc/shadow
   20  passwd user01
   21  sudo passwd user01
   22  sudo tail -1 /etc/shadow
   23  mkdir /home/guest
   24  sudo mkdir /home/guest
   25  sudo useradd -D -b /home/guest
   26  sudo useradd -D
   27  sudo useradd user02
   28  sudo tail -1 /etc/passwd
   29  sudo useradd -u 2000 -g 10 -m -d /home/guest/user03 -s /bin/sh user03
   30  sudo tail -1 /etc/passwd
   31  history
   ```

# 5장
## 파일시스템

```
    3  sudo fdisk /dev/sd
    4  sudo fdisk /dev/sdb
    5  lsblk
    6  sudo partprobe
    7  sudo partprobe /dev/sdb
    8  ls -li /etc/passwd
    9  lsblk
   10  sudo fdisk /dev/sdb
   11  sudo partprobe /dev/sdb
   12  lsblk
   13  sudo mkfs.xfs /dev/sdb1
   14  sudo file -s /dev/sdb1
   15  lsblk
   16  sudo mkswap /dev/sdb2
   17  sudo file -s /dev/sdb2
   18  sudo file -s /dev/sdb1
   19  sudo blkid
   20  ls -l /dev/sdb2
   21  ls -l /dev
   22  sudo mkdir /mnt/xfsdata
   23  sudo mount /dve/sdb1 /mnt/xfsdata
   24  sudo systemctl daemon-reload
   25  df -h
   26  mount
   27  mount | grep sdb1
   28  sudo mount /dev/sdb1 /mnt/xfsdata
   29  df -h
   30  mount | grep sdb1
   31  sudo touch a /mnt/xfsdata
   32  ls -l /dev/sdv1
   33  ls -l /dev/sdb1
   34  ll -f /dev/sdb1
   35  ll /dev/sdb1
   36  sudo touch /mnt/xfsdata/a
   37  ls -l /mnt/xfsdata
   38  sudo vi /etc/fstab
   39  sudo umount /mnt/xfsdata
   40  mount | grep sdb
   41  sudo mount -a
   42  df -h | grep sdb1
   43  sudo mount | grep sd
   44  reboot
   45  sudo reboot
   46  ls -l /mnt/xfsdata
   47  mount | grep sdb
   ```
