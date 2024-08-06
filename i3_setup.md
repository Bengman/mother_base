## Install i3 on Qubes

in dom0:
```sudo qubes-dom0-update i3 i3-settings-qubes```

Log out and change the windows manger from xfxe to i3 in the login screen.
### i3 tweaks

Change backround on dom0 to black:
Create the file .Xresources in dom0 and add:
```
URxvt*background: black
URxvt*foreground: white
```

Create keybindings for i3lock, in .config/i3/config:
```
# keybinding to lock screen
bindsym $mod+Shift+l exec "i3lock -c 000000"
```
(do not forget to check that the binding is not already in use).
