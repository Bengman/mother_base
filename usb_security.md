Source: 
- https://blog.invisiblethings.org/2011/05/31/usb-security-challenges.html
- https://www.qubes-os.org/doc/device-handling-security/#usb-security

First there are all the _physical_ attacks that could be conducted with the help of USB devices. These are generally not so interesting, because if one includes physical attacks in the threat model, then it really opens up lots of possibilities of various attacks, and generally a physical attacker always wins. Still, there are a few very cheap and easy physical attacks that one would like to avoid, or make harder, such as the [Evil Maid Attacks](http://theinvisiblethings.blogspot.com/2009/10/evil-maid-goes-after-truecrypt.html) or the [Cold Boot Attacks](http://citp.princeton.edu/memory/). Strictly speaking these are not problems inherent to USB itself, but rather with lack of Trusted Boot, or OS not cleaning properly secrets from memory upon shutdown. They are just made simple thanks to bootable USB sticks.

Much more interesting USB-related physical attacks are those that take advantage of the specifics of the USB standard. One [example](http://www.blackhat.com/presentations/bh-usa-05/BH_US_05-Barrall-Dewey.pdf) here would be a malicious USB device that exposes intentionally malformed info about itself in order to exploit a potential flaw in a USB Host Controller driver that processes this info upon each new USB device connect. [Or](https://media.blackhat.com/bh-dc-11/Larimer/BlackHat_DC_2011_Larimer_Vulnerabiliters%20w-removeable%20storage-Slides.pdf) a malicious USB device that would trick the OS (Windows at least) into downloading a known buggy USB driver (or even an intentionally malicious driver, legally submitted to WHQL by the attacker) and then exploit the driver.

Another class of physical attacks made possible by the USB specification are malicious USB devices that [pretend to be a keyboard](https://media.blackhat.com/bh-dc-11/Stavrou-Wang/BlackHat_DC_2011_Stavrou_Zhaohui_USB_exploits-Slides.pdf) or mouse. The input devices, such as keyboard, are actually the most security sensitive devices, because an attacker who controls the keyboard can do everything the user can do, which basically means: can do everything, at least with regards to the user's data.  

Finally, the USB, as the names stands, is a _bus_ interconnect, which means all the USB devices sharing the same USB controller are capable of sniffing and spoofing signals on the bus. This is one of the key differences between USB and PCI Express standards, where the latter uses a peer-to-peer interconnect architecture.

Ok, so these all above were _physical_ attacks. Let's now look at, much more fatal, _software_ attacks.

The infamous class of attacks exploiting various autorun or auto-preview behaviors is the most known example, but also the easiest, at least in theory, to protect against.
  

Much more interesting are software attacks that attempt to exploit potential flaws in the USB stacks – similarly like the physical attacks mentioned above, just that this time not requiring any hardware-level modifications to the USB device. Exposing a [malformed partition table](http://www.securityfocus.com/archive/1/516615) is a great example of such an attack. Even if we have all the autorun mechanisms disabled, still, when we're inserting a storage medium the OS _always_ attempts to parse the partition table in order to e.g. create devices symbolizing each partition/volume (e.g. /dev/sdbX devices).

  

Now, this is really a problematic attack, because the malformed partition table can be written onto a fully legitimate USB stick by malware. Imagine e.g. you have two physically separated machines (air-gapped), belonging to two different security domains, and you want to transfer files from one to another. You insert the USB stick into the first machine, copy files, and then insert the stick to the second machine. If the first machine was compromised, it could have altered the partition table on the USB stick, and now when this stick is inserted into the other machine its malformed partition table might exploit a buffer overflow in the code used by the second OS to parse the stick's partition information. Air-gapped systems, huh? We avoid this attack vector in Qubes by using a special inter-domain [file copy mechanism](http://wiki.qubes-os.org/trac/wiki/Qfilecopy) that doesn't require any metadata parsing.

A variation of the above attack would be to expose a malicious file system metadata, but this time the target OS would have to actually mount the partition for the attack to work (and, of course, there would have to be bugs in the OS file system parsing code, although these  seem to be quite common on most OSes).

First we should realize that USB devices, unlike PCI Express devices, _cannot_ be independently delegated to different domains (VMs). This is because IOMMU technologies, such as Intel VT-d, operate only on PCIe device granularity. This means we can only delegate a whole USB _controller_ to a domain, including _all_ of the USB devices connected to this controller/hub.

As show above the connection of an untrusted USB device to dom0 is a security risk since the device can attack an arbitrary USB driver (which are included in the linux kernel), exploit bugs during partition-table-parsing or simply pretend to be a keyboard. There are many ready-to-use implementations of such attacks, e.g. a [USB Rubber Ducky](https://shop.hak5.org/products/usb-rubber-ducky-deluxe). The whole USB stack is put to work to parse the data presented by the USB device in order to determine if it is a USB mass storage device, to read its configuration, etc. This happens even if the drive is then assigned and mounted in another qube.

To avoid this risk, use a USB qube.
The USB qube acts as a secure handler for potentially malicious USB devices, preventing them from coming into contact with dom0 (which could otherwise be fatal to the security of the whole system). It thereby mitigates some of the [security risks of using USB devices](https://www.qubes-os.org/doc/device-handling-security/#usb-security).

Attaching a USB device to a VM (USB passthrough) will **expose your target qube** to most of the security issues associated with the USB-stack. If possible, use a method specific for particular device type for example, block devices, instead of this generic one.