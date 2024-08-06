# Mother_base
A reasonably secure base setup for offensive operations and research.

This set up is based on Qubes OS and heavily focuses on isolation and
compartmentalization. It uses minimal air-gapped vm's for 

It is roughly the set up I have been using for offensive securiity research and 
operations.

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

This will give you the base setup. You will then need to create vm's suited 
for your needs such as Windows development machines, secure communication vm's 
and Kali Linux boxes.
