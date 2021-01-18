---
title: "Raspberry PI pass NTP config with DHCP."
tags:
  - linux
  - raspberry-pi
---

Configure NTP on a Raspberry PI with DHCP option 42 NTP Servers.
DHCP allows to send a whole list of options along with the IP address for the Raspberry PI.

## NTP client

Raspberry PI OS is debian based and uses [systemd-timesyncd](https://wiki.archlinux.org/index.php/systemd-timesyncd) as NTP client. 
The default configuration contains a list of debian pool ntp servers but these require internet access.
This can be seen with the command

```
timedatectl show-timesync --all
```

When you want to run a Raspberry PI in a vlan with no internet access you will need to setup a local NTP server that the Raspberry PI can use.
This could be a local router or a different Raspberry PI (with internet access).

Raspberry PI OS uses [dhcpcd](https://www.daemon-systems.org/man/dhcpcd.8.html) as DHCP client. 
If instead [systemd-networkd](https://wiki.archlinux.org/index.php/systemd-networkd) was used, the NTP config would be passed automatically. 
Another alternative would be to use Gnome [NetworkManager](https://wiki.archlinux.org/index.php/NetworkManager).

## Configure dhcpcd

The first step is to configure dhcpcd to use the option ntp_servers. 
Uncomment the following line in `/etc/dhcpcd.conf`

```
option ntp_servers
```

## Add dhcpcd hook

The next step is to alter the systemd-timesyncd configuration when the DHCP option is received.
dhcpcd allows hook scripts to be configured that can do this.
[dhcpcd-run-hooks](https://www.daemon-systems.org/man/dhcpcd-run-hooks.8.html) are configured in `/lib/dhcpcd/dhcpcd-hooks`.
Add the following hook `/lib/dhcpcd/dhcpcd-hooks/60-timesyncd.conf`

```
add_ntp_conf()
{
    exec > /etc/systemd/timesyncd.conf.d/$interface.conf
    echo "[Time]"
    echo "NTP=$new_ntp_servers"
    systemctl restart systemd-timesyncd
}

remove_ntp_conf() 
{
    rm -f /etc/systemd/timesyncd.conf.d/$interface.conf
    systemctl restart systemd-timesyncd
}

# For ease of use, map DHCP6 names onto our DHCP4 names
case "$reason" in
BOUND6|RENEW6|REBIND6|REBOOT6|INFORM6)
	new_ntp_servers="$new_dhcp6_sntp_servers"
;;
esac

if $if_up; then
	add_ntp_conf
elif $if_down; then
	remove_ntp_conf
fi
```

Now reboot the Raspberry PI and the NTP configuration will be added to `/etc/systemd/timesyncd.conf.d/eth0.conf`. 
You can also check the output of `timedatectl show-timesync --all` to validate that timesyncd is configured correctly. 

