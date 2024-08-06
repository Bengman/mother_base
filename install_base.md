First check out the hardware compability list at https://www.qubes-os.org/hcl/


What you will need:
- Compatible hardware
- Yubikey
- USB-memory stick



1. Download iso from https://www.qubes-os.org/downloads/


2. Very download against the hashsum from https://mirrors.edge.kernel.org/qubes/iso/Qubes-R4.2.1-x86_64.iso.DIGESTS

`echo "a942911a3a4975831324a064f70b34c6965c4e9f6c95afbc531f04d55f947376 *Qubes-R4.2.1-x86_64.iso" | sha256sum --check


3. Copy iso to USB

`sudo dd if=Qubes-R4.2.1-x86_64.iso of=/dev/sdX conv=fsync bs=4M status=progress

4. Boot from USB and follow the installer.


5. When installed do not forget to update all qubes.