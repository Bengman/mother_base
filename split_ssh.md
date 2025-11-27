
https://forum.qubes-os.org/t/split-ssh/19060

![[Pasted image 20240710145913.png]]

# Security Benefits

In the setup described in this guide, even an attacker who manages to gain access to the `ssh-client` VM will not be able to obtain the user’s private key since it is simply not there. Rather, the private key remains in the `vault` VM, which is extremely unlikely to be compromised if nothing is ever copied or transferred into it. In order to gain access to the vault VM, the attacker would require the use of, e.g., a general Xen VM escape exploit or a signed, compromised package which is already installed in the TemplateVM upon which the vault VM is based.

# Overview

1. Make sure the TemplateVM you plan to use is up to date.
2. Create `vault` and `admin-ssh` AppVMs.
3. Create an ssh key in your `vault` AppVM and set up automatic key adding prompt.
4. Set up VM interconnection



# Creating AppVMs

 `qvm-create --template=debian-13-minimal --label=green --property=memory=400 --property=vcpus=1 admin-ssh`
 
 `qvm-prefs admin-ssh netvm sys-vpn`
 

# Setting up SSH

Install `ssh-askpass` in the template of your `vault` VM. It will be used by `ssh-agent` to ask for confirmation, for keys added using `ssh-add -c`.

For Debian templates:  

`sudo apt-get install ssh-askpass-gnome socat`

Perform the next steps in the AppVM `vault`.

Generate an SSH key pair.
`ssh-keygen` 


Make a new directory in your vault appvm `~/.config/autostart`

- ```bash
    mkdir -p ~/.config/autostart
    ```
    
- Create the file `~/.config/autostart/ssh-add.desktop`
    
    - Open the file with e.g. `gedit`
        
- ```bash
    gedit ~/.config/autostart/ssh-add.desktop
    ```
    
- Paste the following contents:
    

```
[Desktop Entry]
Name=ssh-add
Exec=ssh-add -c
Type=Application
```

# Setting Up VM Interconnection

### In `dom0`:

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
        


### In the Template of Your AppVM `vault`:

We now need to write a small script that handles connection requests from `ssh-client` and forwards them to the SSH agent in your `vault`. As we are using qrexec as communication method we have to place it in a special location and have to name it just as the policy file we just created in `dom0`.

1. Create and edit `/etc/qubes-rpc/qubes.SshAgent`.
    
    - Open the file with e.g. `gedit`
        
        `sudo gedit ****/etc/qubes-rpc/qubes.SshAgent`
        
    - Paste the following contents:
    ``` 
    #!/bin/sh 
        # Qubes App Split SSH Script  
        
        # safeguard - Qubes notification bubble for each ssh request notify-send "[$(qubesdb-read /name)] SSH agent access from: $QREXEC_REMOTE_DOMAIN"  
        
        # SSH connection socat - "UNIX-CONNECT:$SSH_AUTH_SOCK"
    ```
        
        
1. Make it executable
    

`sudo chmod +x /etc/qubes-rpc/qubes.SshAgent`

## In the AppVM `admin-ssh`

Theoretically, you can use SSH in any AppVM. However, if you are considering split-SSH as an additional security layer it is probably reasonable to also think about which VMs you will be using SSH in. For instance, you might want a dedicated `admin` domain for these purposes. Depending on how many systems you plan to access and where they are located, it could also be preferable to have different VMs with different firewall rules for Intranet and Internet administration.

We want to make sure that our `ssh-client` is prepared to use split-ssh right after the VM has started. Therefore, we add a script in `rc.local` (Which will run at VM startup) to listen for responses from `vault` and make SSH use this connection by modifying the user’s `.bashrc`.

1. Edit `/rw/config/rc.local`.
    
    - Open the file with e.g. `gedit`.
        
        `sudo gedit /rw/config/rc.local`
        
    - Add the following to the bottom of the file:
      
```
# SPLIT SSH CONFIGURATION >>> # replace "vault" with your AppVM name which stores the ssh private key(s) 
SSH_VAULT_VM="vault"  

if [ "$SSH_VAULT_VM" != "" ]; then
	export SSH_SOCK="/home/user/.SSH_AGENT_$SSH_VAULT_VM"   
	rm -f "$SSH_SOCK"
	sudo -bengamu user /bin/sh -c "umask 177 && exec socat 'UNIX-LISTEN:$SSH_SOCK,fork' 'EXEC:qrexec-client-vm $SSH_VAULT_VM qubes.SshAgent'" & 
	fi 
	# <<< SPLIT SSH CONFIGURATION`
```    
2. Edit `~/.bashrc` and add the following to the bottom of the file:
    
    - Open the file with e.g. `gedit`
        
        `gedit ~/.bashrc`
        
    - Add the following to the bottom of the file:
        

```
# SPLIT SSH CONFIGURATION >>> # replace "vault" with your AppVM name which stores the ssh private key(s) 
SSH_VAULT_VM="vault"  

if [ "$SSH_VAULT_VM" != "" ]; then   
 export SSH_AUTH_SOCK="/home/user/.SSH_AGENT_$SSH_VAULT_VM" 
fi # <<< SPLIT SSH CONFIGURATION
```
   
Shutdown the template and restart your `vault` VM.


With this configuration you’ll be prompted for entering your password every time you start your vault VM to be able to make use of your SSH key.

### Configuring qube firewall

We want to restrict traffic to only port 22

in dom0

`qvm-firewall admin-ssh del --rue-no 0`

`qvm-firewall admin-ssh add accept dstports=22 proto=tcp`