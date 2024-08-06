
Reference: https://forum.qubes-os.org/t/split-ssh/19060

![](https://github.com/Qubes-Community/Contents/raw/master/attachment/wiki/split-ssh/diagram.svg)

# Security Benefits

In the setup described in this guide, even an attacker who manages to gain access to the `ssh-client` VM will not be able to obtain the user’s private key since it is simply not there. Rather, the private key remains in the `vault` VM, which is extremely unlikely to be compromised if nothing is ever copied or transferred into it. In order to gain access to the vault VM, the attacker would require the use of, e.g., a general Xen VM escape exploit or a signed, compromised package which is already installed in the TemplateVM upon which the vault VM is based.

# Overview

1. Make sure the TemplateVM you plan to use is up to date.
2. Create `vault` and `ssh-client` AppVMs.
3. Create an ssh key in your `vault` AppVM and set up automatic key adding prompt.
4. Set up VM interconnection
5. (Strongly Encouraged) Create a KeePassXC Database and set up SSH Agent Integration in KeePassXC.

# [](https://forum.qubes-os.org/t/split-ssh/19060#preparing-your-system-3)Preparing your system

Make sure the templates you plan to base your AppVMs on are [up-to-date 2](https://www.qubes-os.org/doc/software-update-domu/#updating-software-in-templatevms).

# [](https://forum.qubes-os.org/t/split-ssh/19060#creating-appvmshttpswwwqubes-osorgdocgetting-startedadding-removing-and-listing-qubes-4)[Creating AppVMs](https://www.qubes-os.org/doc/getting-started/#adding-removing-and-listing-qubes)

If you’ve installed Qubes OS using the default options, a few qubes including a vault AppVM has been created for you. Skip the first step if you don’t wish to create another vault.

1. Create a new vault AppVM (`vault`) based on your chosen template. Set networking to `(none)`.
    
    ![vault creation](https://forum.qubes-os.org/uploads/db3820/original/2X/7/7fd53ab2f6b6d09c6f23a0d438f9052820960bf0.png)
    
2. Create a SSH Client AppVM (`ssh-client`). This VM will be used to make SSH connections to your remote machine.
    
    ![ssh-client creation](https://forum.qubes-os.org/uploads/db3820/original/2X/5/5cfff3e1b16f0b8a36de66882082bdf8c1526564.png)
    

# [](https://forum.qubes-os.org/t/split-ssh/19060#setting-up-ssh-5)Setting up SSH

Install `ssh-askpass` in the template of your `vault` VM. It will be used by `ssh-agent` to ask for confirmation, for keys added using `ssh-add -c`.

For Fedora templates:  

 `sudo dnf install openssh-askpass`

For Debian templates:  

`sudo apt-get install ssh-askpass-gnome`

Perform the next steps in the AppVM `vault`.

1. Generate an SSH key pair. Skip this step if you already have your keys. Note that it is _okay_ to not enter a password for your private keys since the `vault` AppVM has no networking. If you still want to encrypt your keys you must refer to the [Securing Your Private Key](https://forum.qubes-os.org/t/split-ssh/19060#securing-your-private-key) section.
    
    `[user@vault ~]$ ssh-keygen -t ed25519 -a 500 Generating public/private ed25519 key pair. Enter file in which to save the key (/home/user/.ssh/id_ed25519):  Created directory '/home/user/.ssh'. Enter passphrase (empty for no passphrase):  Enter same passphrase again:  Your identification has been saved in /home/user/.ssh/id_ed25519 Your public key has been saved in /home/user/.ssh/id_ed25519.pub The key fingerprint is: SHA256:DBxSxZcp16d1NSVSid3m8HRipUDM2INghQ4Sx3jPEDo user@vault The key's randomart image is: +--[ED25519 256]--+ |    o==+++.@++o=*| |    o==o+ B BoOoB| |    Eoo* +   *.O.| |     . o+   .   o| |        S        | |                 | |                 | |                 | |                 | +----[SHA256]-----+`
    
    **-t**: type  
    **-a**: num_trials  
    
    Please note that the key fingerprint and the randomart image will differ.
    
    For more information about `ssh-keygen`, run `man ssh-keygen`.
    

**Notice:** Skip the following steps if you plan on using KeePassXC.

2. Make a new directory `~/.config/autostart`
    
    `mkdir -p ~/.config/autostart`
    
3. Create the file `~/.config/autostart/ssh-add.desktop`
    
    - Open the file with e.g. `gedit`
        
        `gedit ~/.config/autostart/ssh-add.desktop`
        
    - Paste the following contents:
        
        `[Desktop Entry] Name=ssh-add Exec=ssh-add -c Type=Application`
        
    
    **Note:** If you’ve specified a custom name for your key using _-f_, you should adjust `Exec=ssh-add` to `Exec=ssh-add <path-to-your-key-file>`.
    

# [](https://forum.qubes-os.org/t/split-ssh/19060#setting-up-vm-interconnection-6)Setting Up VM Interconnection

## [](https://forum.qubes-os.org/t/split-ssh/19060#in-dom0-7)In `dom0`:

To control which VM is allowed as a client, which may act as the server and how we want this interaction to happen, we have to write a policy file for qrexec in `dom0`.

1. Create and edit `/etc/qubes/policy.d/50-ssh.policy`.
    
    - Open the file with e.g. `nano`.
        
        `sudo nano /etc/qubes/policy.d/50-ssh.policy`
        
    - If you want to explicitly allow only this connection, add the following line:
        
        `qubes.SshAgent * ssh-client vault ask target=vault`
        
    - If you want all of your VMs to potentially be an `ssh-client` or a `vault`, add the following line:
        
        `qubes.SshAgent * @anyvm @anyvm ask`
        
    - If you want the input field to be “prefilled” by your `vault` VM, append `target=vault` so it looks like for example:
        
        `qubes.SshAgent * @anyvm @anyvm ask target=vault`
        

 Old policy file for Qubes R4.1

**Note:** There are many ways to fine-tune this policy. For more details see the [Qubes qrexec documentation 23](https://www.qubes-os.org/doc/qrexec/#policy-files).

## [](https://forum.qubes-os.org/t/split-ssh/19060#in-the-template-of-your-appvm-vault-8)In the Template of Your AppVM `vault`:

We now need to write a small script that handles connection requests from `ssh-client` and forwards them to the SSH agent in your `vault`. As we are using qrexec as communication method we have to place it in a special location and have to name it just as the policy file we just created in `dom0`.

1. Create and edit `/etc/qubes-rpc/qubes.SshAgent`.
    
    - Open the file with e.g. `gedit`
        
        `sudo gedit /etc/qubes-rpc/qubes.SshAgent`
        
    - Paste the following contents:
    ``` 
    #!/bin/sh 
        # Qubes App Split SSH Script  
        
        # safeguard - Qubes notification bubble for each ssh request notify-send "[$(qubesdb-read /name)] SSH agent access from: $QREXEC_REMOTE_DOMAIN"  
        
        # SSH connection socat - "UNIX-CONNECT:$SSH_AUTH_SOCK"
    ```
        
        
1. Make it executable
    

`sudo chmod +x /etc/qubes-rpc/qubes.SshAgent`

## [](https://forum.qubes-os.org/t/split-ssh/19060#in-the-appvm-ssh-client-9)In the AppVM `ssh-client`

Theoretically, you can use SSH in any AppVM. However, if you are considering split-SSH as an additional security layer it is probably reasonable to also think about which VMs you will be using SSH in. For instance, you might want a dedicated `admin` domain for these purposes. Depending on how many systems you plan to access and where they are located, it could also be preferable to have different VMs with different firewall rules for Intranet and Internet administration.

We want to make sure that our `ssh-client` is prepared to use split-ssh right after the VM has started. Therefore, we add a script in `rc.local` (Which will run at VM startup) to listen for responses from `vault` and make SSH use this connection by modifying the user’s `.bashrc`.

1. Edit `/rw/config/rc.local`.
    
    - Open the file with e.g. `gedit`.
        
        `sudo gedit /rw/config/rc.local`
        
    - Add the following to the bottom of the file:
        
        `# SPLIT SSH CONFIGURATION >>> # replace "vault" with your AppVM name which stores the ssh private key(s) SSH_VAULT_VM="vault"  if [ "$SSH_VAULT_VM" != "" ]; then   export SSH_SOCK="/home/user/.SSH_AGENT_$SSH_VAULT_VM"   rm -f "$SSH_SOCK"   sudo -u user /bin/sh -c "umask 177 && exec socat 'UNIX-LISTEN:$SSH_SOCK,fork' 'EXEC:qrexec-client-vm $SSH_VAULT_VM qubes.SshAgent'" & fi # <<< SPLIT SSH CONFIGURATION`
        
2. Edit `~/.bashrc` and add the following to the bottom of the file:
    
    - Open the file with e.g. `gedit`
        
        `gedit ~/.bashrc`
        
    - Add the following to the bottom of the file:
        
        `# SPLIT SSH CONFIGURATION >>> # replace "vault" with your AppVM name which stores the ssh private key(s) SSH_VAULT_VM="vault"  if [ "$SSH_VAULT_VM" != "" ]; then   export SSH_AUTH_SOCK="/home/user/.SSH_AGENT_$SSH_VAULT_VM" fi # <<< SPLIT SSH CONFIGURATION`
        

# [](https://forum.qubes-os.org/t/split-ssh/19060#securing-your-private-key-10)Securing Your Private Key

Although passwords wouldn’t protect you against a full system compromise (attacker could place a keylogger), it’s possible for an adversary to gain read-only access to some of your files (e.g., file shares or offline backups of data). This becomes even more likely if you plan to also use your data outside of Qubes and not be able to modify anything. Passwords are advisable for mitigating these threats .

You can either [use the built-in password utility](https://forum.qubes-os.org/t/split-ssh/19060#using-the-built-in-password-utility) of your private key combined with a graphical prompt or prefer to [use KeePassXC](https://forum.qubes-os.org/t/split-ssh/19060#using-keepassxc). Please note that since `ssh-askpass` prompt is displayed on `vault` VM boot, it is not possible to use both configurations simultaneously.

## [](https://forum.qubes-os.org/t/split-ssh/19060#using-the-built-in-password-utility-and-ssh-askpass-11)Using the Built-in Password Utility and `ssh-askpass`

You should have added `ssh-askpass` to your vault template earlier when [setting up SSH](https://forum.qubes-os.org/t/split-ssh/19060#setting-up-ssh).

1. Either add a password to an existing private key with `ssh-keygen -p` or directly create a key pair with a password (enter password when prompted during the creation process, see [above](https://forum.qubes-os.org/t/split-ssh/19060#setting-up-ssh)). Note that the location and name of your private key may differ.
    
    `[user@vault ~]$ ssh-keygen -p  Enter file in which the key is (/home/user/.ssh/id_rsa): /home/user/.ssh/id_ed25519 Key has comment 'user@vault' Enter new passphrase (empty for no passphrase):  Enter same passphrase again:  Your identification has been saved with the new passphrase.`
    
2. Shutdown the template and restart your `vault` VM.
    

With this configuration you’ll be prompted for entering your password every time you start your vault VM to be able to make use of your SSH key.
