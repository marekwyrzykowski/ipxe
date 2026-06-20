# ipxe


# initrd.rtl8169.gz

```bash
mkdir /tmp/a
wget https://deb.debian.org/debian/dists/trixie/main/installer-amd64/current/images/netboot/debian-installer/amd64/linux -O /tmp/a/vmlinuz
wget https://deb.debian.org/debian/dists/trixie/main/installer-amd64/current/images/netboot/debian-installer/amd64/initrd.gz -O /tmp/a/initrd.gz
mkdir -p /tmp/rebuild_initrd
cd /tmp/rebuild_initrd
zcat /tmp/a/initrd.gz | cpio -idmv
#wget https://marekwyrzykowski.github.io/ipxe/debian/preseed.cfg -O preseed.cfg
mkdir -p lib/firmware/rtl_nic
cd lib/firmware/rtl_nic
wget https://git.kernel.org/pub/scm/linux/kernel/git/firmware/linux-firmware.git/plain/rtl_nic/rtl8168h-2.fw
cd /tmp/rebuild_initrd
find . | cpio -H newc -o | gzip -9 > /tmp/a/initrd.rtl8169.gz
```


# usb.ipxe

```bash
dd if=/dev/zero of=/home/marek/usb.ipxe.img bs=1M count=100
sudo parted /home/marek/usb.ipxe.img mklabel gpt
sudo parted /home/marek/usb.ipxe.img mkpart "primary" fat32 1MiB 100%
sudo parted /home/marek/usb.ipxe.img set 1 boot on
sudo losetup -P -f /home/marek/usb.ipxe.img 
sudo mkfs.vfat -F 32 -n "NETBOOT" $(losetup -a | grep "/home/marek/usb.ipxe.img" | cut -d":" -f1)p1
sudo mkdir -p /tmp/usb.ipxe
sudo mount $(losetup -a | grep "/home/marek/usb.ipxe.img" | cut -d":" -f1)p1 /tmp/usb.ipxe/
sudo mkdir -p /tmp/usb.ipxe/EFI/BOOT
sudo wget https://boot.ipxe.org/x86_64-efi/snponly.efi -O /tmp/usb.ipxe/EFI/BOOT/BOOTX64.EFI
sudo wget https://marekwyrzykowski.github.io/ipxe/usb.ipxe/autoexec.ipxe -O /tmp/usb.ipxe/EFI/BOOT/autoexec.ipxe
sudo umount /tmp/usb.ipxe 
sudo losetup -d $(losetup -a | grep "/home/marek/usb.ipxe.img" | cut -d":" -f1)
```
