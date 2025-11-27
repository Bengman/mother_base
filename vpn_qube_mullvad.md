For this VPN set up I will use Mullvads Wireguard setup. Their guide can be found here:
https://mullvad.net/en/help/wireguard-on-qubes-os

You could off course use any VPN, such as a corporate VPN configuration as well.

## Create a ProxyVM

`qvm-create --template=debian-13-xfce --label=green --property=memory=800 --property=vcpus=2 sys-vpn

`qvm-prefs sy-vpn provides_network true`

`qvm-prefs sy-vpn autostart true`

## Download a WireGuard configuration

In another AppVM (not MullvadVPN) that you use for web surfing:

1. Open a web browser and log in to our [WireGuard configuration file generator](https://mullvad.net/account/wireguard-config/).
2. Select **Linux** as the platform and then click on **Generate key**.
3. Select a country, a city and a server.
4. Click on **Download file**.
5. Click on the Qubes app menu and go to your current AppVM and open **Files**.
6. Open the Downloads folder and right click on the downloaded WireGuard file.
7. Select **Copy To Other AppVM...** and then enter **MullvadVPN** as the Target and click on **OK**. The MullvadVPN ProxyVM will start when you do this.

## Install wireguard in template

`sudo apt install wireguard-tools`

On Debian you also need resolvconf
`sudo apt install resolvconf`



## Configure WireGuard

Click on the Qubes app menu and go to **Service: MullvadVPN** and open the Terminal.

Copy the WireGuard .conf file that was copied from the other AppVM to the /home/user/ folder:  
`cp /home/user/QubesIncoming/*/se-got-wg-001.conf /home/user/`
ln -s /usr/bin/resolvectl /usr/local/bin/resolvconf
#### Enable autostart of WireGuard

1. Edit the /rw/config/rc.local file using a text editor like nano:  
    `sudo dnf install nano -y`
2. Run `sudo nano /rw/config/rc.local`
3. Add `wg-quick up /home/user/se-got-wg-001.conf` (or the config file you copied) on a new line in the bottom.
4. Press Ctrl+O (Enter) and then Ctrl+X to save and exit.

#### Make sure that WireGuard connects

1. Run `sudo wg-quick up /home/user/se-got-wg-001.conf`
2. Run `ping 10.64.0.1`
3. Run `curl https://am.i.mullvad.net/connected`
4. Run `sudo wg` and check for a WireGuard network interface and a peer handshake.


