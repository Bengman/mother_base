
By default Qubes has two protection mechanisms against attackers. The first is full disk encryption and the second the user login screen / lockscreen. 
This guide is focusing on adding multi-factor authentication to the second one.

# Setup login with YubiKey

To use the YubiKey for multi-factor authentication we need to

- install software for the YubiKey.
- configure the YubiKey for the [Challenge-Response](https://en.wikipedia.org/wiki/Challenge%E2%80%93response_authentication) mode.
- store the password for YubiKey Login and the Challenge-Response in dom0,
- enable YubiKey authentication for every service you want to use it for.

All these requirements are described below, step by step, for the YubiKey.

1. Install YubiKey software in the template on which your USB VM is based.

For Debian.
    
    ```
     sudo apt-get install yubikey-personalization ykman
    ```
Note: If you are using disposable template for your sys-usb then the packages must be installed in the disposable template.

2. Install [qubes-app-yubikey](https://github.com/QubesOS/qubes-app-yubikey) in dom0. This provides the program to authenticate with password and YubiKey.
    
    ```
     sudo qubes-dom0-update qubes-yubikey-dom0
    ```
    
3. Make sure the Yubikey is connected

```
ykman list
YubiKey 5 NFC (5.2.6) [OTP+FIDO+CCID] Serial: 12095270
```

Configure your YubiKey for challenge-response mode by generating a new key in the first slot:

`ykman otp chalresp --generate 1 --touch


4. Paste your `AESKEY` into `/etc/qubes/yk-keys/yk-secret-key.hex` in dom0.
    
    
5. You need to define a new password that is only used in combination with the YubiKey . You can write this password in plain text into `/etc/qubes/yk-keys/login-pass` in dom0. This is considered safe as dom0 is ultimately trusted anyway.
    
6. To enable multi-factor authentication for a service, you need to add
    
    ```
     auth include yubikey
    ```
    
    to the corresponding service file in `/etc/pam.d/` in dom0. This means, if you want to enable the login via YubiKey for xscreensaver (the default screen lock program), you add the line at the beginning of `/etc/pam.d/xscreensaver`. If you want to use the login for a tty shell, add it to `/etc/pam.d/login`. Add it to `/etc/pam.d/lightdm` if you want to enable the login for the default display manager and so on.
    
    It is important, that `auth include yubikey` is added at the beginning of these files, otherwise it will most likely not work.

So to enable the yubikey in i3 add it to the following services:
- `/etc/pam.d/lightdm`
- `/etc/pam.d/i3lock`

4. Adjust the USB VM name in case you are using something other than the default `sys-usb` by editing `/etc/qubes/yk-keys/vm` in dom0.
5. Check so that the slot is configured correctly in `yk-slot`file as well.


#### Usage

When you want to authenticate

1. plug your YubiKey into an USB slot,
2. enter the password associated with the YubiKey,
3. press Enter and
4. press the button of the YubiKey, if you configured the confirmation (it will light up or blink).

When everything is ok, your screen will be unlocked.

In any case you can still use your normal login password, but do it in a secure location where no one can snoop your password.

#### Optional: Enforce YubiKey  Login
Edit `/etc/pam.d/yubikey` (or appropriate file if you are using other screen locker program) and remove `default=ignore` so the line looks like this.

```
auth [success=done] pam_exec.so expose_authtok quiet /usr/bin/yk-auth
```

#### Optional: Locking the screen when YubiKey is removed (i3 edition)
You can setup your system to automatically lock the screen when you unplug your YubiKey. This will require creating a simple qrexec service which will expose the ability to lock the screen to your USB VM, and then adding a udev hook to actually call that service.

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