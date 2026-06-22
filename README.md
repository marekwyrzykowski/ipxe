# ipxe


# initrd.rtl8169.gz

```bash
mkdir /tmp/a
wget https://deb.debian.org/debian/dists/trixie/main/installer-amd64/current/images/netboot/debian-installer/amd64/linux -O /tmp/a/vmlinuz
wget https://deb.debian.org/debian/dists/trixie/main/installer-amd64/current/images/netboot/debian-installer/amd64/initrd.gz -O /tmp/a/initrd.gz
mkdir -p /tmp/rebuild_initrd
cd /tmp/rebuild_initrd
zcat /tmp/a/initrd.gz | cpio -idmv
wget https://raw.githubusercontent.com/marekwyrzykowski/ipxe/refs/heads/main/debian/preseed.cfg -O preseed.cfg
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
sudo wget https://raw.githubusercontent.com/marekwyrzykowski/ipxe/refs/heads/main/usb.ipxe/autoexec.ipxe -O /tmp/usb.ipxe/EFI/BOOT/autoexec.ipxe
sudo umount /tmp/usb.ipxe 
sudo losetup -d $(losetup -a | grep "/home/marek/usb.ipxe.img" | cut -d":" -f1)
```


# initrd.rtl8169.gz - v2

```bash
#!/bin/bash
set -e

# 1. Przygotowanie czystego instalatora Debian 13 (Trixie
rm -rf /tmp/a /tmp/rebuild_initrd /tmp/udeb_packages

mkdir -p /tmp/a
wget https://deb.debian.org/debian/dists/trixie/main/installer-amd64/current/images/netboot/debian-installer/amd64/linux -O /tmp/a/vmlinuz
wget https://deb.debian.org/debian/dists/trixie/main/installer-amd64/current/images/netboot/debian-installer/amd64/initrd.gz -O /tmp/a/initrd.gz

# 2. Rozpakowanie ramdysku initrd
mkdir -p /tmp/rebuild_initrd
cd /tmp/rebuild_initrd
zcat /tmp/a/initrd.gz | cpio -idmv

# # 3. Dodanie sterowników (Firmware) Realtek
# mkdir -p lib/firmware/rtl_nic
# cd lib/firmware/rtl_nic
# wget https://git.kernel.org/pub/scm/linux/kernel/git/firmware/linux-firmware.git/plain/rtl_nic/rtl8168h-2.fw

# 4. Pobieranie pakietów udeb dla HTTPS oraz KLIENTA CZASU RDATE
mkdir -p /tmp/udeb_packages
cd /tmp/udeb_packages

wget https://ftp.debian.org/debian/pool/main/c/ca-certificates/ca-certificates-udeb_20260601_all.udeb
wget https://ftp.debian.org/debian/pool/main/o/openssl/libcrypto4-udeb_4.0.1-1_amd64.udeb
wget https://ftp.debian.org/debian/pool/main/o/openssl/libssl4-udeb_4.0.1-1_amd64.udeb
wget https://ftp.debian.org/debian/pool/main/r/rdate/rdate-udeb_1.11-3_amd64.udeb
wget https://deb.debian.org/debian/pool/main/libb/libbsd/libbsd0-udeb_0.12.2-3_amd64.udeb
wget https://ftp.debian.org/debian/pool/main/libm/libmd/libmd0-udeb_1.2.0-2_amd64.udeb


# 5. Wypakowanie pakietów bezpośrednio do struktury initrd
dpkg-deb -x ca-certificates-udeb_20260601_all.udeb /tmp/rebuild_initrd/
dpkg-deb -x libcrypto4-udeb_4.0.1-1_amd64.udeb /tmp/rebuild_initrd/
dpkg-deb -x libssl4-udeb_4.0.1-1_amd64.udeb /tmp/rebuild_initrd/
dpkg-deb -x rdate-udeb_1.11-3_amd64.udeb /tmp/rebuild_initrd/
dpkg-deb -x libbsd0-udeb_0.12.2-3_amd64.udeb /tmp/rebuild_initrd/
dpkg-deb -x libmd0-udeb_1.2.0-2_amd64.udeb /tmp/rebuild_initrd/


# ==========================================================================
# Dodanie portów do /etc/services
# ==========================================================================
mkdir -p /tmp/rebuild_initrd/etc
cat << 'EOF' >> /tmp/rebuild_initrd/etc/services
time            37/tcp
time            37/udp
EOF

# 6. Uporządkowanie i naprawa dowiązań symbolicznych w /lib/
cd /tmp/rebuild_initrd/lib/

if [ -f libcrypto.so.4 ] && [ ! -L libcrypto.so.4 ]; then
    mv libcrypto.so.4 libcrypto.so.4.0.1
    ln -sf libcrypto.so.4.0.1 libcrypto.so.4
fi

if [ -f libssl.so.4 ] && [ ! -L libssl.so.4 ]; then
    mv libssl.so.4 libssl.so.4.0.1
    ln -sf libssl.so.4.0.1 libssl.so.4
fi

# 7. Pakowanie nowego, zmodyfikowanego obrazu initrd
cd /tmp/rebuild_initrd
find . | cpio -H newc -o | gzip -9 > /tmp/a/initrd.rtl8169.gz

# Czyszczenie śmieci
rm -rf /tmp/udeb_packages /tmp/rebuild_initrd

```
