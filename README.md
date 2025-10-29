# Mother Base

![](https://static.wikia.nocookie.net/metalgear/images/a/af/2132411934_view.jpg)

# Table of Contents
- [Step 1 - Base setup with hardware encryption](install_base.md)
- [Step 2 - Set up MFA with Yubikey](yubikey_setup.md)
- [Step 3 - Set up Split-GPG](split_gpg.md)
- [Step 4 - Minimal air gapped pass vault using split-gpg](minimal_pass_vault.md)
- [Step 5 - Securing data in transit using a VPN Qube](vpn_qube_mullvad.md) 
- [Step 6 - Creating a secure administrative host with Split-SSH](split_ssh.md)
- [Step 7 - Secure communications]()
- [Step 8 - Windows 11 templates]()
- [Step 9 - Pentesting templates]()
- [Security considerations]()
    - [Hypervisor security considerations](hypervisor_security.md)
    - [USB security considerations](usb_security.md)
    - [Security considerations of updating systems](update_security.md)
    - [Security considerations of passwords, GPG-keys and SSH-keys](keys.md)
    - [Clipboard security considerations](clipboard_security.md)
    - [Storage security considerations](storage_security.md)
    - [Security considerations sharing files between vms](file_security.md)
    - [Network security considerations](network_security.md)


I’ve been using Qubes as my main OS in my role as pentester/red teamer for over 6 years now.

As an offensive security consultant we are often connecting our laptops to a large number of unkown and customer networks, we are running potentially untrusted offensive security 
tooling and at the same time handling very sensitive customer data.
In my opinion using a (often corporate) Windows host with a simple Kali virtual machine does not cut it when it comes to protecting sensitive customer data and managing risks during engagements.

This set up is based on [Qubes OS](https://www.qubes-os.org/intro/) and heavily focuses on isolation and
compartmentalization. It will give you a solid base to work from using some of Qubes most 
powerfull features. 

Building this setup “manually” using kvm for example can take junior pentesters weeks, however following my guide will get you up and running in a day or two.
This guides main focus is security and not necessarily privacy, however you will get a large portion of privacy built-in with this setup.

I want to give  massive credits to the exceptional Qubes OS project, for making this OS and all the documentation behind it. 

![](https://www.qubes-os.org/attachment/site/qubes-trust-level-architecture.png)

I have also tired so summerize the most important security considerations that are behind the design of the 
set-up.
