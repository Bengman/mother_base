Qubes architecture implements secure storage by having a separate, unprivileged, domain which has ac-
cess to the disk and other mass storage devices, and that also hosts the Xen block backend that are used to
export the virtual block devices to all other VMs, including the Dom0. Cryptography is used to assure that the
storage domain cannot compromise the filesystems used by other domains in any meaningful way, which
makes the storage domain security non-critical. This means that if the attacker managed to compromise the
storage domain, this would not automatically let the attacker to compromise any other VM.

Unlike in case of the Network domain, the Storage domain does not contain any world-facing code, so the
need to create an isolated domain for “disk drivers” might seem a bit of an overkill at first look. However, we
should remember that there is lots of code running to provide all the subsystem services: block device back-
ends, file exchange daemon (described earlier), USB PV backends, etc. By isolating all this code in a sepa-
rate, unprivileged domain, we donʼt need to worry about potential bugs that might be present in this code.
Indeed, Qubes architecture assures, that even if the attacker compromised the storage domain, this doesnʼt
automatically let the attacker to compromise the rest of the system, including the VMs that have their filesys-
tem provided via the Storage domain.
The property of the storage domain being security non-critical is achieved with the help of three mecha-
nisms:
1. Encrypted per-VM private storage devices (only given AppVM and Dom0 know the key)
2. Signed block device with root filesystem used by AppVMs (only the UpdateVM and Dom0 know the sign-
ing key)
3. TPM-based trusted/verified boot process implemented with the help of Intel TXT

