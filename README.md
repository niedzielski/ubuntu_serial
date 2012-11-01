Copyright 2012 Stephen Niedzielski. Licensed under GPLv3.

## Recipe for Creating a Serial Installer
1. Download the latest [Ubuntu Server LTS daily](http://cdimage.ubuntu.com/ubuntu-server/precise/daily/current/).
1. `iso=precise-server-amd64.iso`
1. `mkdir iso_r`
1. `sudo mount "$iso" iso_r`
1. `cp -a iso_r iso_rw`
1. `sudo umount iso_r`
1. `rmdir iso_r`
1. Edit iso_rw/boot/grub/grub.cfg.
  1. Under the "Install Ubuntu Server" entry, append: `console=ttyS0,115200n8`
  1. Before the first `menuentry` add:
    1. `serial --unit=0 --speed=115200 --word=8 --parity=no --stop=1`
    1. `terminal --timeout=2 serial console`
1. Edit iso_rw/isolinux/isolinux.cfg.
  1. Comment out: `#ui gfxboot bootlogo`
  1. Append: `serial 0 115200 0×003`
1. Edit iso_rw/isolinux/txt.cfg.
  1.  Under the "Install Ubuntu Server" entry, append: `console=ttyS0,115200n8`
1. `sudo genisoimage -input-charset utf-8 -r -V 'Ubuntu Server' -cache-inodes -l -b isolinux/isolinux.bin -c isolinux/boot.cat -no-emul-boot -boot-load-size 4 -boot-info-table -o "$iso" iso_rw`
1. `sudo chown $(id -u):$(id -g) "$iso"`
1. `sudo rm -rf iso_rw`

## Installation on VM
Note: Ubuntu Server 12.04 LTS consumes about 1.8 GB. I recommend a 4 GB install.

![](https://raw.github.com/niedzielski/ubuntu_serial/master/docs/2012-10-07-14-00-08-195673682.png) <br />
![](https://raw.github.com/niedzielski/ubuntu_serial/master/docs/2012-10-07-14-00-14-806935120.png) <br />
![](https://raw.github.com/niedzielski/ubuntu_serial/master/docs/2012-10-07-14-00-20-519735547.png) <br />
![](https://raw.github.com/niedzielski/ubuntu_serial/master/docs/2012-10-07-14-00-25-515467043.png) <br />
![](https://raw.github.com/niedzielski/ubuntu_serial/master/docs/2012-10-07-14-00-30-186250303.png) <br />
![](https://raw.github.com/niedzielski/ubuntu_serial/master/docs/2012-10-07-14-00-38-750833651.png) <br />
![](https://raw.github.com/niedzielski/ubuntu_serial/master/docs/2012-10-07-14-02-23-396185117.png) <br />
![](https://raw.github.com/niedzielski/ubuntu_serial/master/docs/2012-10-07-14-02-24-641066033.png) <br />

1. `socat UNIX-CONNECT:/tmp/precise_server_serial_0_pipe PTY,link=/tmp/precise_server_serial_0_pty&`
1. `echo stty cols $COLUMNS rows $LINES` (Copy and paste this after logging into the VM.)
1. `screen /tmp/precise_server_serial_0_pty 115200`

Note: `sudo adduser "$USER" dialout`

## Updating the Grub
1. Edit /etc/default/grub.
  1. `GRUB_CMDLINE_LINUX_DEFAULT="console=ttyS0,115200n8"`
  1. `GRUB_CMDLINE_LINUX="console=ttyS0,115200n8"`
  1. `GRUB_TERMINAL=serial`
  1. `GRUB_SERIAL_COMMAND="serial --speed=115200 --unit=0 --word=8 --parity=no --stop=1"`
1. Write the changes to /boot/grub/grub.cfg: `sudo update-grub`.
  1. `sudo reboot 0`.

See also: `info -f grub`

## Convert to a Real HDD
1. Insert the disk.
1. `mount` to find the disk.
1. `sudo umount #/media/...`
1. Ensure it's hosed `sudo mkfs.vfat -I #/dev/sdX` to prevent misconceptions.
1. `time vboxmanage clonehd --format RAW ~/VirtualBox\ VMs/precise_server_serial_0/Snapshots/\{0295b898-a4c5-415a-805e-69b70d00feb9\}.vdi ~/work/ubuntu_serial/precise_server_serial_0.img`. This took about two 13 seconds. Note: avoid the pitfall of grabbing the root VDI. You can also write straight to `/dev/sdX`. If all works well, you should see a file that's exactly 2 GB. Note: don't use `qemu-img convert`. It's appears to be busted!
1. `time sudo dd if=precise_server_serial_0.img #of=/dev/sdX`. This took about 18 minutes to copy over a USB to SD converter on a class 10 card and about 27 and a half minutes on an unmarked card, and about five and half minutes over a USB to SATA converter. If all works well, you should see some parititons in `gparted`.

## Boot QEMU from a Real HDD
`sudo qemu-system-x86_64 -enable-kvm -m 512 #/dev/sdX`