# Mother_base
A reasonably secure base setup for offensive security operations and research.


This set up is based on [Qubes OS](https://www.qubes-os.org/intro/) and heavily focuses on isolation and
compartmentalization. It will give you a solid base to work from using some of Qubes most 
powerfull features. 

I want to give  massive credits to the exceptional Qubes OS project, for making this OS and all the documentation behind it. 


![](https://www.qubes-os.org/attachment/site/qubes-trust-level-architecture.png)

![](http://2.bp.blogspot.com/-llpRcqX8oBw/ToQpuLMy_HI/AAAAAAAAAJo/3pZ1S-qHupA/s1600/qubes-adv-config.png)

The above images are from the Qubes project and will give you an idea of how a set up could look. 

# Table of Contents
- [Base setup](install_base.md)
- [Set up i3](i3_setup.md)
- [Set up Yubikey](yubikey_setup.md)
- [Minimal pass vault](minimal_pass_vault.md)
- [Split-GPG](split_gpg.md)
- [Split-SSH](split_ssh.md)
- [VPN Qube](vpn_qube_mullvad.md)
- [Security considerations]()
    - [Hypervisor security considerations](hypervisor_security.md)
    - [USB security considerations](usb_security.md)
    - [Security considerations of updating systems](update_security.md)
    - [Security considerations of passwords, GPG-keys and SSH-keys](keys.md)
    - [Clipboard security considerations](clipboard_security.md)
    - [Storage security considerations](storage_security.md)
    - [Security considerations sharing files between vms](file_security.md)
    - [Network security considerations](network_security.md)

Use disposable vm's for as much as possible such as browsing the web and handle untrusted
files.

While this will give you the base setup, you will then need to tailor it to your needs including creating vm's
such as Windows development machines, secure communication vm's 
and Kali linux boxes.

I have also tired so summerize the most important security considerations that are behind the design of the 
set-up.
