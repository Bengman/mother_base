https://mullvad.net/sv/help/qubes-os-4-and-mullvad-vpn

Create a new qube

First install the Debian 12 template (if you do not already have it) using the following command in the Terminal Emulator (dom0):

sudo qubes-dom0-update qubes-template-debian-12

Click on the Qubes app menu > Qubes Tools > Create Qubes VM.

    Name and label: MullvadVPN.
    Type: AppVM (persistent home, volatile root).
    Template: debian-12 (or later).
    Networking: default (sys-firewall).
    Click on the Advanced tab and check (enable) Provides network access to other qubes.
    Click on OK.

The newly created MullvadVPN ProxyVM qube will show up as "Service: MullvadVPN" in the Qubes app menu and not "Qube: MullvadVPN" due to its "provides network" setting.
Download an OpenVPN configuration

In another AppVM (not MullvadVPN) that you use for web surfing:

    Open a web browser and log in to our OpenVPN configuration file generator.
    Select Linux as the platform.
    Select Sweden as the country and Gothenburg or Malmö or Stockholm as the city.
    Click on Download zip archive.
    Open the Downloads folder and right click on the downloaded OpenVPN file and select Extract Here.
    Click on the extracted folder and select Copy To Other AppVM... and then enter MullvadVPN as the Target and click on OK. The MullvadVPN ProxyVM will start when you do this.

Install OpenVPN

Install OpenVPN in the Debian-12 template so your MullvadVPN ProxyVM can use that.

    Click on the Qubes app menu and go to Template: debian-12 and open the Terminal.
    In the Terminal run the command sudo apt install openvpn -y
    Disable the OpenVPN service: sudo systemctl disable openvpn.service
    Shut down the VM with the command sudo shutdown -h now
    Click on the Qubes app menu and go to Qubes Tools > Qube Manager.
    In Qube Manager, restart the MullvadVPN ProxyVM (so that OpenVPN is added to it).

Set the Network

    In Qube Manager, select the AppVM that you want to use with the MullvadVPN ProxyVM and click on the Stop button in the toolbar to shut it down.
    Right click on the same AppVM and select Settings.
    On the Basic tab, click on the Net qube drop-down list and select MullvadVPN.
    Click on OK.
    Click on the Start button in the toolbar to start the AppVM again.

Configure OpenVPN

    Click on the Qubes app menu and go to Service: MullvadVPN and open the Terminal.
    Now you will extract and copy the OpenVPN configurations files that were copied from the other AppVM to the /rw/config/vpn folder.
    Create the folder:
    sudo mkdir /rw/config/vpn
    In the Terminal, change directory to where the configurations file are located and copy the files to /rw/config/vpn.
    sudo cp /home/user/QubesIncoming/*/*/*/* /rw/config/vpn
    Set execute permissions:
    sudo chmod 755 /rw/config/vpn/update-resolv-conf

Test that OpenVPN connects

    Run sudo su and cd /rw/config/vpn and openvpn --config mullvad_xx_xxx.conf (use the config file you copied). You should see "Initialization Sequence Completed" on one of the last lines.
    Run curl https://am.i.mullvad.net/connected in a new Terminal. It should say that you are connected to Mullvad.
    Press Ctrl+c on the keyboard in the OpenVPN window to disconnect it.

Enable autostart of OpenVPN

    Edit the /rw/config/rc.local file using a text editor. First install nano:
    sudo apt install nano -y
    Then run sudo nano /rw/config/rc.local
    Add openvpn --cd /rw/config/vpn --config mullvad_xx_xxx.conf --daemon (use the config file you copied) on a new line in the bottom.
    Press Ctrl+O (Enter) and then Ctrl+X to save and exit.
    In Qube Manager, restart the MullvadVPN ProxyVM.
    Click on the Qubes app menu and go to Service: MullvadVPN and open the Terminal.
    Run curl https://am.i.mullvad.net/connected. It should say that you are connected to Mullvad.

Add DNS hijacking rules

Now we will add firewall rules to redirect DNS requests to 10.8.0.1 (the DNS on the VPN server) for all AppVMs that use the MullvadVPN ProxyVM.

Make sure that you have started an AppVM that has the Networking set to MullvadVPN, otherwise the "vif" IP address will not be visible.

Still in the MullvadVPN ProxyVM Terminal:

    To find out your vif* IP address, run ip a | grep -i vif. Write down the "inet" address that you get, for example 10.137.0.47.
    Edit the firewall user file with nano:
    sudo nano /rw/config/qubes-firewall-user-script
    Copy the text in the text area below and paste it in the bottom of your file.
    Replace 10.137.0.47 with your own vif* IP address that you got before.

# replace 10.137.0.47 with the IP address of your vif* interface
virtualif=10.137.0.47
vpndns1=10.8.0.1
iptables -F OUTPUT
iptables -I FORWARD -o eth0 -j DROP
iptables -I FORWARD -i eth0 -j DROP
iptables -t mangle -A FORWARD -p tcp --tcp-flags SYN,RST SYN -j TCPMSS --clamp-mss-to-pmtu
iptables -F PR-QBS -t nat
iptables -A PR-QBS -t nat -d $virtualif -p udp --dport 53 -j DNAT --to $vpndns1
iptables -A PR-QBS -t nat -d $virtualif -p tcp --dport 53 -j DNAT --to $vpndns1

    5. Press Ctrl+O (Enter) and then Ctrl+X to save and exit.
Add qube firewall rules

In Qube Manger, select MullvadVPN then right click and select Settings.

Make the following changes:

    Make sure it is still set to use sys-firewall as "Networking".
    Check Start qube automatically on boot.
    Click on the Firewall rules tab.
    Click on Limit outgoing internet connections to ....
    Click on + and enter the IP addresses of the VPN servers that you want to be able to connect to. You can find it in the OpenVPN configuration file (mullvad_xx_xxx.conf) on the "remote" lines, or in our Servers list.
    Click on OK.

If you connect to Sweden or the Netherlands then you can add the following IP ranges:

        45.83.220.0/24  Sweden (Malmö)
      141.98.255.0/24  Sweden (Malmö)
    193.138.218.0/24  Sweden (Malmö)
    185.213.152.0/24  Sweden (Helsingborg)
    185.213.154.0/24  Sweden (Gothenburg)
      185.65.135.0/24  Sweden (Stockholm)
      185.65.134.0/24  Netherlands (Amsterdam)
