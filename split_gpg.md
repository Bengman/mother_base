

This set up will use PGP multiple encryption jobs, such as encrypting passwords in pass, e-mails etc.

This guide will show how to use Qubes [split-GPG](https://www.qubes-os.org/doc/split-gpg/) feature to securely handle encrypting and decrypting stuff using a air-gapped private PGP-key.

![[Pasted image 20240710145419.png]]

## Advantages of Split GPG vs. traditional GPG with a smart card
It is often thought that the use of smart cards for private key storage guarantees ultimate safety. While this might be true (unless the attacker can find a usually-very-expensive-and-requiring-physical-presence way to extract the key from the smart card) but only with regards to the safety of the private key itself. However, there is usually nothing that could stop the attacker from requesting the smart card to perform decryption of all the user documents the attacker has found or need to decrypt. In other words, while protecting the user’s private key is an important task, we should not forget that ultimately it is the user data that are to be protected and that the smart card chip has no way of knowing the requests to decrypt documents are now coming from the attacker’s script and not from the user sitting in front of the monitor. (Similarly the smart card doesn’t make the process of digitally signing a document or a transaction in any way more secure – the user cannot know what the chip is really signing. Unfortunately this problem of signing reliability is not solvable by Split GPG.)

With Qubes Split GPG this problem is drastically minimized, because each time the key is to be used the user is asked for consent (with a definable time out, 5 minutes by default), plus is always notified each time the key is used via a tray notification from the domain where GPG backend is running. This way it would be easy to spot unexpected requests to decrypt documents.

# Configuring Split GPG using minimal templates

I like using minimal templates to remove all preinstalled bloatware and reduce the attack surface of the templates.

1. Download Debian minimal template
```sudo qubes-dom0-update qubes-template-debian-13-minimal```

2. Configure passwordless root in template
```[user@dom0 ~]$ qvm-run -u root <DISTRO_NAME>-<RELEASE_NUMBER>-minimal xterm```

```apt install qubes-core-agent-passwordless-root```

(and for you who just jumped out of their chairs: https://www.qubes-os.org/doc/vm-sudo/)


3. install required tools in template
```sudo apt install qubes-gpg-split```


4. In dom0, make sure the qubes-gpg-split-dom0 package is installed.
```sudo qubes-dom0-update qubes-gpg-split-dom0```


### Setting up the GPG backend domain

First, create a dedicated app qube for storing your keys. It is recommended that this domain be network disconnected (set its netvm to `none`) and only used for this one purpose. 

```qvm-create --template=debian-11-minimal --label=black gpg-vault```

also set their netvm to none
```qvm-prefs gpg-vault netvm nonecompability```

Make sure that gpg is installed there. At this stage you can add the private keys you want to store there, or you can now set up Split GPG and add the keys later. 

This is pretty much all that is required. However, you might want to modify the default timeout: this tells the backend for how long the user’s approval for key access should be valid. (The default is 5 minutes.) You can change this via the `QUBES_GPG_AUTOACCEPT` environment variable. You can override it e.g. in `~/.profile`:

```
[user@gpg-vault ~]$ echo "export QUBES_GPG_AUTOACCEPT=86400" >> ~/.profile
```


Please be aware of the caveat regarding passphrase-protected keys in the [Current limitations](https://www.qubes-os.org/doc/split-gpg/#current-limitations) section.


