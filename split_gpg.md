https://www.qubes-os.org/doc/split-gpg/

![[Pasted image 20240710145419.png]]

## Configuring Split GPG[](https://www.qubes-os.org/doc/split-gpg/#configuring-split-gpg)

In dom0, make sure the `qubes-gpg-split-dom0` package is installed.

```
[user@dom0 ~]$ sudo qubes-dom0-update qubes-gpg-split-dom0
```

Make sure you have the `qubes-gpg-split` package installed in the template you will use for the GPG domain.

For Debian or Whonix:

```
[user@debian-10 ~]$ sudo apt install qubes-gpg-split
```

For Fedora:

```
[user@fedora-32 ~]$ sudo dnf install qubes-gpg-split
```

### Setting up the GPG backend domain[](https://www.qubes-os.org/doc/split-gpg/#setting-up-the-gpg-backend-domain)

First, create a dedicated app qube for storing your keys (we will be calling it the GPG backend domain). It is recommended that this domain be network disconnected (set its netvm to `none`) and only used for this one purpose. In later examples this app qube is named `work-gpg`, but of course it might have any other name.

Make sure that gpg is installed there. At this stage you can add the private keys you want to store there, or you can now set up Split GPG and add the keys later. To check which private keys are in your GPG keyring, use:

```
[user@work-gpg ~]$ gpg -K
/home/user/.gnupg/secring.gpg
-----------------------------
sec   4096R/3F48CB21 2012-11-15
uid                  Qubes OS Security Team <security@qubes-os.org>
ssb   4096R/30498E2A 2012-11-15
(...)
```

This is pretty much all that is required. However, you might want to modify the default timeout: this tells the backend for how long the user’s approval for key access should be valid. (The default is 5 minutes.) You can change this via the `QUBES_GPG_AUTOACCEPT` environment variable. You can override it e.g. in `~/.profile`:

```
[user@work-gpg ~]$ echo "export QUBES_GPG_AUTOACCEPT=86400" >> ~/.profile
```

Please note that previously, this parameter was set in ~/.bash_profile. This will no longer work. If you have the parameter set in ~/.bash_profile you _must_ update your configuration.

Please be aware of the caveat regarding passphrase-protected keys in the [Current limitations](https://www.qubes-os.org/doc/split-gpg/#current-limitations) section.

### Configuring the client apps to use Split GPG backend[](https://www.qubes-os.org/doc/split-gpg/#configuring-the-client-apps-to-use-split-gpg-backend)

Normally it should be enough to set the `QUBES_GPG_DOMAIN` to the GPG backend domain name and use `qubes-gpg-client` in place of `gpg`, e.g.:

```
[user@work-email ~]$ export QUBES_GPG_DOMAIN=work-gpg
[user@work-email ~]$ gpg -K
[user@work-email ~]$ qubes-gpg-client -K
/home/user/.gnupg/secring.gpg
-----------------------------
sec   4096R/3F48CB21 2012-11-15
uid                  Qubes OS Security Team <security@qubes-os.org>
ssb   4096R/30498E2A 2012-11-15
(...)

[user@work-email ~]$ qubes-gpg-client secret_message.txt.asc
(...)
```

Note that running normal `gpg -K` in the demo above shows no private keys stored in this app qube.

A note on `gpg` and `gpg2`:

Throughout this guide, we refer to `gpg`, but note that Split GPG uses `gpg2` under the hood for compatibility with programs like Enigmail (which now supports only `gpg2`). If you encounter trouble while trying to set up Split GPG, make sure you’re using `gpg2` for your configuration and testing, since keyring data may differ between the two installations.

### Advanced Configuration[](https://www.qubes-os.org/doc/split-gpg/#advanced-configuration)

The `qubes-gpg-client-wrapper` script sets the `QUBES_GPG_DOMAIN` variable automatically based on the content of the file `/rw/config/gpg-split-domain`, which should be set to the name of the GPG backend VM. This file survives the app qube reboot, of course.

```
[user@work-email ~]$ sudo bash
[root@work-email ~]$ echo "work-gpg" > /rw/config/gpg-split-domain
```

Split GPG’s default qrexec policy requires the user to enter the name of the app qube containing GPG keys on each invocation. To improve usability for applications like Thunderbird with Enigmail, in `dom0` place the following line at the top of the file `/etc/qubes-rpc/policy/qubes.Gpg`:

```
work-email  work-gpg  allow
```

where `work-email` is the Thunderbird + Enigmail app qube and `work-gpg` contains your GPG keys.

You may also edit the qrexec policy file for Split GPG in order to tell Qubes your default gpg vm (qrexec prompts will appear with the gpg vm preselected as the target, instead of the user needing to type a name in manually). To do this, append `default_target=<vmname>` to `ask` in `/etc/qubes-rpc/policy/qubes.Gpg`. For the examples given on this page:

```
@anyvm  @anyvm  ask default_target=work-gpg
```

Note that, because this makes it easier to accept Split GPG’s qrexec authorization prompts, it may decrease security if the user is not careful in reviewing presented prompts. This may also be inadvisable if there are multiple app qubes with Split GPG set up.