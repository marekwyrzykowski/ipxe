# ipxe


# initrd.rtl8169.gz

```bash
mkdir /tmp/a
cd /tmp/a
wget https://deb.debian.org/debian/dists/trixie/main/installer-amd64/current/images/netboot/debian-installer/amd64/linux -O vmlinuz
wget https://deb.debian.org/debian/dists/trixie/main/installer-amd64/current/images/netboot/debian-installer/amd64/initrd.gz
mkdir -p /tmp/rebuild_initrd
cd /tmp/rebuild_initrd
zcat /tmp/a/initrd.gz | cpio -idmv
mkdir -p lib/firmware/rtl_nic
cd lib/firmware/rtl_nic
wget https://git.kernel.org/pub/scm/linux/kernel/git/firmware/linux-firmware.git/plain/rtl_nic/rtl8168h-2.fw
cd /tmp/rebuild_initrd
find . | cpio -H newc -o | gzip -9 > /tmp/a/initrd.rtl8169.gz
```
