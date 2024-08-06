https://www.qubes-os.org/doc/mfa/#setup-login-with-yubikey--nitrokey3

#### Setup login with YubiKey / NitroKey3[](https://www.qubes-os.org/doc/mfa/#setup-login-with-yubikey--nitrokey3)

To use the YubiKey / NitroKey3 for multi-factor authentication you need to

- install software for the YubiKey / NitroKey3,
- configure the YubiKey for the [Challenge-Response](https://en.wikipedia.org/wiki/Challenge%E2%80%93response_authentication) mode or the NitroKey3 for [HOTP](https://en.wikipedia.org/wiki/HMAC-based_one-time_password) mode,
- store the password for YubiKey / NitroKey3 Login and the Challenge-Response / HOTP secret in dom0,
- enable YubiKey / NitroKey3 authentication for every service you want to use it for.

All these requirements are described below, step by step, for the YubiKey and NitroKey3. Note that setting up both a YubiKey and a NitroKey3 is not supported.

1. Install YubiKey / NitroKey3 software in the template on which your USB VM is based. Without this software the challenge-response / HOTP mechanism won’t work.
    
    **YubiKey**
    
    For Fedora.
    
    ```
     sudo dnf install ykpers
    ```
    
    For Debian.
    
    ```
     sudo apt-get install yubikey-personalization
    ```

2. Install [qubes-app-yubikey](https://github.com/QubesOS/qubes-app-yubikey) in dom0. This provides the program to authenticate with password and YubiKey / NitroKey3.
    
    ```
     sudo qubes-dom0-update qubes-yubikey-dom0
    ```
    
3. Configure your YubiKey / NitroKey3:
    
    **YubiKey**
    
    Configure your YubiKey for challenge-response `HMAC-SHA1` mode. This can be done on any qube, e.g. a disposable (you need to [attach the YubiKey](https://www.qubes-os.org/doc/how-to-use-usb-devices/) to this app qube though) or directly on the sys-usb vm.
    
    You need to (temporarily) install the package “yubikey-personalization-gui” and run it by typing `yubikey-personalization-gui` in the command line.
    
    - In the program go to `Challenge-Response`,
    - select `HMAC-SHA1`,
    - choose `Configuration Slot 2`,
    - optional: enable `Require user input (button press)` (recommended),
    - use `fixed 64 bit input` for `HMAC-SHA1 mode`,
    - insert the YubiKey (if not done already) and make sure that it is attached to the vm,
    - press `Write Configuration` once you are ready.

4. Paste your `AESKEY` into `/etc/qubes/yk-keys/yk-secret-key.hex` in dom0. Note that if you had previously used a NitroKey3 with this package, you _must_ delete the file `/etc/qubes/yk-keys/nk-hotp-secret` or its content!
    
    
5. As mentioned before, you need to define a new password that is only used in combination with the YubiKey / NitroKey3. You can write this password in plain text into `/etc/qubes/yk-keys/login-pass` in dom0. This is considered safe as dom0 is ultimately trusted anyway.
    
    However, if you prefer you can paste a hashed password instead into `/etc/qubes/yk-keys/login-pass-hashed.hex` in dom0.
    
    You can calculate your hashed password using the following two commands. First run the following command to store your password in a temporary variable `password`. (This way your password will not leak to the terminal command history file.)
    
    ```
     read -r password
    ```
    
    Now run the following command to calculate your hashed password.
    
    ```
     echo -n "$password" | openssl dgst -sha1 | cut -f2 -d ' '
    ```
    
6. To enable multi-factor authentication for a service, you need to add
    
    ```
     auth include yubikey
    ```
    
    (same for YubiKey and NitroKey3) to the corresponding service file in `/etc/pam.d/` in dom0. This means, if you want to enable the login via YubiKey / NitroKey3 for xscreensaver (the default screen lock program), you add the line at the beginning of `/etc/pam.d/xscreensaver`. If you want to use the login for a tty shell, add it to `/etc/pam.d/login`. Add it to `/etc/pam.d/lightdm` if you want to enable the login for the default display manager and so on.
    
    It is important, that `auth include yubikey` is added at the beginning of these files, otherwise it will most likely not work.
    
4. Adjust the USB VM name in case you are using something other than the default `sys-usb` by editing `/etc/qubes/yk-keys/vm` in dom0.
    

#### Usage[](https://www.qubes-os.org/doc/mfa/#usage)

When you want to authenticate

1. plug your YubiKey / NitroKey3 into an USB slot,
2. enter the password associated with the YubiKey / NitroKey3,
3. press Enter and
4. press the button of the YubiKey / NitroKey3, if you configured the confirmation (it will light up or blink).

When everything is ok, your screen will be unlocked.

In any case you can still use your normal login password, but do it in a secure location where no one can snoop your password.

#### Optional: Enforce YubiKey / NitroKey3 Login[](https://www.qubes-os.org/doc/mfa/#optional-enforce-yubikey--nitrokey3-login)

Edit `/etc/pam.d/yubikey` (or appropriate file if you are using other screen locker program) and remove `default=ignore` so the line looks like this.

```
auth [success=done] pam_exec.so expose_authtok quiet /usr/bin/yk-auth
```

#### Optional: Locking the screen when YubiKey / NitroKey3 is removed[](https://www.qubes-os.org/doc/mfa/#optional-locking-the-screen-when-yubikey--nitrokey3-is-removed)

You can setup your system to automatically lock the screen when you unplug your YubiKey / NitroKey3. This will require creating a simple qrexec service which will expose the ability to lock the screen to your USB VM, and then adding a udev hook to actually call that service.

In dom0:

1. First configure the qrexec service. Create `/etc/qubes-rpc/custom.LockScreen` with a simple command to lock the screen. In the case of xscreensaver (used in Xfce) it would be:
    
    ```
    DISPLAY=:0 xscreensaver-command -lock
    ```
    
2. Then make `/etc/qubes-rpc/custom.LockScreen` executable.
    
    ```
    sudo chmod +x /etc/qubes-rpc/custom.LockScreen
    ```
    
3. Allow your USB VM to call that service. Assuming that it’s named `sys-usb` it would require creating `/etc/qubes-rpc/policy/custom.LockScreen` with:
    
    ```
    sys-usb dom0 allow
    ```
    

In your USB VM:

1. Create udev hook. Store it in `/rw/config` to have it persist across VM restarts. For example name the file `/rw/config/yubikey.rules`. Add the following line:
    
    ```
    ACTION=="remove", SUBSYSTEM=="usb", ENV{ID_SECURITY_TOKEN}=="1", RUN+="/usr/bin/qrexec-client-vm dom0 custom.LockScreen"
    ```
    
2. Ensure that the udev hook is placed in the right place after VM restart. Append to `/rw/config/rc.local`:
    
    ```
    ln -s /rw/config/yubikey.rules /etc/udev/rules.d/
    udevadm control --reload
    ```
    
3. Then make `/rw/config/rc.local` executable.
    
    ```
    sudo chmod +x /rw/config/rc.local
    ```
    
4. For changes to take effect, you need to call this script manually for the first time.
    
    ```
    sudo /rw/config/rc.local
    ```
    

If you use KDE, the command(s) in first step would be different:

```
# In the case of USB VM being autostarted, it will not have direct access to D-Bus
# session bus, so find its address manually:
kde_pid=`pidof kdeinit4`
export `cat /proc/$kde_pid/environ|grep -ao 'DBUS_SESSION_BUS_ADDRESS=[[:graph:]]*'`
qdbus org.freedesktop.ScreenSaver /ScreenSaver Lock
```