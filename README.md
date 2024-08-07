# Mother_base
A reasonably secure base setup for offensive security operations and research.

This set up is based on [Qubes OS](https://www.qubes-os.org/intro/) and heavily focuses on isolation and
compartmentalization. It will give you a solid base to work from using some of Qubes most 
powerfull features. 

![](https://www.qubes-os.org/attachment/site/qubes-trust-level-architecture.png)

![](http://2.bp.blogspot.com/-llpRcqX8oBw/ToQpuLMy_HI/AAAAAAAAAJo/3pZ1S-qHupA/s1600/qubes-adv-config.png)

The above images are from the Qubes project and will give you an idea of how a set up could look. 

# Table of Contents
1. [Base setup](install_base.md)
2. [Set up i3](i3_setup.md)
3. [Set up Yubikey](yubikey_setup.md)
4. [Minimal pass vault](minimal_pass_vault.md)
5. [Split-GPG](split_gpg.md)
6. [Split-SSH](split_ssh.md)
7. [VPN Qube](vpn_qube_mullvad.md)


Use disposable vm's for as much as possible such as browsing the web and handle untrusted
files.

While this will give you the base setup, you will then need to tailor it to your needs including creating vm's
such as Windows development machines, secure communication vm's 
and Kali linux boxes.
