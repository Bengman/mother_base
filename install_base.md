# Hardware compability

Qubes might run into issues on specific hardware. So first off check out the hardware compability list at https://www.qubes-os.org/hcl/
and make sure your hardware is supported correctly.


# Installing Qubes with a bootable USB stick

## Downloading Qubes

1. Go to the official Qubes web site and download ISO from https://www.qubes-os.org/downloads/


![[img/qubes_download.png]]
## Verifying signatures

The Qubes OS Project uses [digital signatures](https://en.wikipedia.org/wiki/Digital_signature) to guarantee the authenticity and integrity of certain important assets. This page explains how to verify those signatures. It is extremely important for your security to understand and apply these practices.

The process consists of:
- Downloading and verifying the Qubes Master Signing Key(QMSK)
- Downloading and verifying the release signing key (RSK)
- Verify the cryptographic hash values of Qubes ISO
### How to import and authenticate the Qubes Master Signing Key

If you’re on Qubes OS, it’s available in every qube ([except dom0](https://github.com/QubesOS/qubes-issues/issues/2544)):

`gpg2 --import /usr/share/qubes/qubes-master-key.asc

If you’re on Debian, it may already be included in your keyring.
Fetch it with GPG:

`gpg2 --fetch-keys https://keys.qubes-os.org/keys/qubes-master-signing-key.asc

Once you’ve obtained the QMSK, you must verify that it’s fingerprint is authentic rather than a forgery. It is strongly suggested to obtain the fingerprint from _multiple independent sources in several different ways_, then comparing the strings of letters and numbers to make sure they match.

```
gpg2 --edit-key 0x427F11FD0FAA4B080123F01CDDFA1A3E36879494
gpg (GnuPG) 1.4.18; Copyright (C) 2014 Free Software Foundation, Inc.
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.

pub  4096R/36879494  created: 2010-04-01  expires: never       usage: SC
                     trust: unknown       validity: unknown
[ unknown] (1). Qubes Master Signing Key

gpg> fpr
pub   4096R/36879494 2010-04-01 Qubes Master Signing Key
Primary key fingerprint: 427F 11FD 0FAA 4B08 0123  F01C DDFA 1A3E 3687 9494

gpg> trust
pub  4096R/36879494  created: 2010-04-01  expires: never       usage: SC
                     trust: unknown       validity: unknown
[ unknown] (1). Qubes Master Signing Key

Please decide how far you trust this user to correctly verify other users' keys

   1 = I don't know or won't say
   2 = I do NOT trust
   3 = I trust marginally
   4 = I trust fully
   5 = I trust ultimately
   m = back to the main menu

Your decision? 5
Do you really want to set this key to ultimate trust? (y/N) y

pub  4096R/36879494  created: 2010-04-01  expires: never       usage: SC
                     trust: ultimate      validity: unknown
[ unknown] (1). Qubes Master Signing Key
Please note that the shown key validity is not necessarily correct
unless you restart the program.
```

## How to import and authenticate release signing keys

Every Qubes OS release is signed by a **release signing key (RSK)**, which is, in turn, signed by the Qubes Master Signing Key (QMSK).

The filename pattern for RSKs is `qubes-release-X-signing-key.asc`, where `X` is either a major or minor Qubes release number.

Fetch it with GPG:
`gpg2 --keyserver-options no-self-sigs-only,no-import-clean --fetch-keys https://keys.qubes-os.org/keys/qubes-release-R4.3.0-signing-key.asc

Now that you have the correct RSK, you simply need to verify that it is signed by the QMSK:
```
gpg2 --check-signatures "Qubes OS Release 4.3 Signing Key"
pub   rsa4096 2024-04-10 [SC]
      F3FA3F99D6281F7B3A3E5E871C3D9B627F3FADA4
uid           [  full  ] Qubes OS Release 4.3 Signing Key
sig!3        1C3D9B627F3FADA4 2024-04-10  Qubes OS Release 4.3 Signing Key
sig!         DDFA1A3E36879494 2024-04-16  Qubes Master Signing Key

gpg: 2 good signatures
```

## How to verify the cryptographic hash values of Qubes ISOs

Download the signature file from https://mirrors.edge.kernel.org/qubes/iso/Qubes-R4.3.0-rc3-x86_64.iso.DIGESTS

![[img/qubes_signatures.png]]
`wget https://mirrors.edge.kernel.org/qubes/iso/Qubes-R4.3.0-rc3-x86_64.iso.DIGESTS`

Verify that the file is legit:
```
gpg2 -v --verify Qubes-R4.3.0-rc3-x86_64.iso.DIGESTS 
gpg: armor header: Hash: SHA256
gpg: original file name=''
gpg: Signature made Sun 26 Oct 2025 01:42:32 PM CET
gpg:                using RSA key F3FA3F99D6281F7B3A3E5E871C3D9B627F3FADA4
gpg: using pgp trust model
gpg: Good signature from "Qubes OS Release 4.3 Signing Key" [full]
gpg: textmode signature, digest algorithm SHA256, key algorithm rsa4096
```

And lastly verify the hash of the .iso

```
sha512sum -c Qubes-R4.3.0-rc3-x86_64.iso.DIGESTS
Qubes-R4.3.0-rc3-x86_64.iso: OK
sha512sum: WARNING: 20 lines are improperly formatted
```

## Preparing the USB-stick
Once we have Downloaded and verified the ISO correctly we can copy the iso onto the USB-memory using the `dd` command.

`sudo dd if=ubes-R4.3.0-rc3-x86_64.iso of=/dev/sdX conv=fsync bs=4M status=progress

Now we can boot from USB-memory and follow the installer.

After the initial install is done you are asked to reboot the system. After that you are presented with 
another customisation screen where you can choose which templates should be installed etc.

For this purpose I choose to not install any additional qubes, just the debian-13 template.
You can also choose if you want to enable USB-keyboards and mice.

*When installed do not forget to update all qubes.

# Optional: Install i3
in dom0: `sudo qubes-dom0-update i3 i3-settings-qubes

Log out and change the windows manger from XFCE to i3 in the login screen top right corner.

### i3 tweaks
Change backround on dom0 to black: Create the file .Xresources in dom0 and add:

`URxvt*background: black 
`URxvt*foreground: white


Create keybindings for i3lock, in .config/i3/config:

```
# keybinding to lock screen
bindsym $mod+Shift+l exec "i3lock -c 000000"
(do not forget to check that the binding is not already in use).
```
