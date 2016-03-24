# ansible-freebsdvps
Building on the work from https://github.com/sinner-/freebsdfun
this is an ansible project to manage a VM based FreeBSD VPS.

## Goals
* Manage PF based firewall.
* Manage OpenVPN based VPN.
  * Provide VPN default gateway functionality.
* TODO: Manage server jails:
  * squid web proxy jail.
  * BIND DNS jail.
  * kerberos auth jail.
  * LDAP directory jail.
  * LDAP uses kerberos.
  * TODO: postfix MTA jail.
    * TODO: postfix to use kerberos+LDAP.
  * nginx jail.
  * TODO: calendar jail.
* TODO: Manage LDAP entries and kerberos accounts for users.
* jail sendmails forward to host root mail.

## First Run
* Boot a FreeBSD 10 VM (tested on FreeBSD 10.2)
* Clone a copy of this repository:
  * `git clone https://github.com/sinner-/ansible-freebsdvps`
  * `cd ansible-freebsdvps`
* Remove .git directory (optional):
  * `rm -rf .git`
* Copy hosts.example to hosts and change the example IP to your VMs IP:
  * `cp hosts.example hosts`
  * `sed -i 's/1.1.1.1/YOUR_VM_IP/' hosts`
* Generate OpenSSL certificates for OpenVPN and copy to:
  * `roles/openvpn/templates/certificates/cacert.pem`
  * `roles/openvpn/templates/certificates/openvpncert.pem`
  * `roles/openvpn/templates/certificates/openvpnkey.pem`
  * `roles/openvpn/templates/certificates/openvpndh4096.pem`
  * `roles/openvpn/templates/certificates/tls-auth.pem`
  * Make sure you have also generated and signed client certificates.
* Run the playbook with the "initialise" tag:
  * `ansible-playbook -i hosts site.yml --tags initialise`.

## Rerun
* Once first run is complete, the SSH port is only accessible via VPN.
* Change the hosts file to point to the VPN server IP:
  * `sed -i 's/YOUR_VM_IP/10.8.0.1/'`
    * (assuming you haven't changed the vpn_tun_subnet variable in group_vars/master)
* Configure your OpenVPN client and connect to the VM.
* Now you can run the playbook with no tags:
  * `ansible-playbook -i hosts site.yml`.

## Adding a new jail
* Edit `group_vars/master` to add the jail name to the jails list.
* Edit `roles/jails/templates/network.j2` to add a new IP address to the bridge for the jail.
* Edit `roles/jails/templates/jail.conf.j2` and add a new jail entry.
  * Use the IP address from prior step.
* Edit `roles/bind/templates/dns.local.j2` to add a forward DNS entry for the jail.
* Edit `roles/bind/templates/reversedns.local.j2` to add a reverse DNS entry for the jail.
* If necessary, edit `roles/firewall/templates/pf.conf.j2` for appropriate firewall rules.
* Make sure the new role for the jail installs a resolv.conf that points to local DNS:
  * `nameserver 192.168.254.11`
  * `search local`
* Run playbook
* On master, run:
  * `export http_proxy="http://squid:3128`
  * `freebsd-update -b /jailhome/jailname fetch`
  * `freebsd-update -b /jailhome/jailname install`
  * `pkg -j jailname update`
