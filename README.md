# ansible-freebsdvps
Building on the work from https://github.com/sinner-/freebsdfun this is an ansible project to manage a VM based FreeBSD VPS.

## Goals
* Manage PF based firewall.
* Manage OpenVPN based VPN.
  * Provide VPN default gateway functionality.
* Manage postfix MTA.
* Manage server jails:
  * squid web proxy jail.
  * BIND DNS jail.
  * nginx jail.
  * radicale CalDAV jail.
  * openldap LDAP jail.
  * mail jail:
    * postfix MTA.
* Unified authentication:
  * radicale uses LDAP.
  * mail jail uses LDAP.

## Bootstrap
* Boot a FreeBSD 11 VM (tested on FreeBSD 11.1)
* Clone a copy of this repository:
  * `git clone https://github.com/sinner-/ansible-freebsdvps`
  * `cd ansible-freebsdvps`
* Remove .git directory (optional):
  * `rm -rf .git`
* Copy hosts.example to hosts and change the example IP to your VMs IP:
  * `cp hosts.example hosts`
  * `sed -i 's/1.1.1.1/YOUR_VM_IP/' hosts`
* Copy host_vars/master.example to host_vars/master
  * `cp host_vars/master.example host_vars/master`
* Generate OpenSSL certificates for OpenVPN and copy to:
  * `roles/openvpn/templates/certificates/cacert.pem`
  * `roles/openvpn/templates/certificates/openvpncert.pem`
  * `roles/openvpn/templates/certificates/openvpnkey.pem`
  * `roles/openvpn/templates/certificates/openvpndh4096.pem`
  * `roles/openvpn/templates/certificates/tls-auth.pem`
  * Make sure you have also generated and signed client certificates.
* Run the playbook with the "bootstrap" tag:
  * `ansible-playbook -i hosts bootstrap.yml`.
* It's not required, but the following set of commands are recommended
  * SSH into the bootstrapped VM with `ssh root@10.8.0.1`.
  * `freebsd-update fetch`
  * `freebsd-update install`
  * `reboot`

## Normal runs
* Once first run is complete, the SSH port is only accessible via VPN.
* Change the hosts file to point to the VPN server IP:
  * `sed -i 's/YOUR_VM_IP/10.8.0.1/' hosts`
    * (assuming you haven't changed the vpn_tun_subnet variable in host_vars/master)
* Configure your OpenVPN client and connect to the VM.
* Now you can run the playbook with no tags:
  * `ansible-playbook -i hosts site.yml`.
* After a normal run is complete you should use `slappasswd` to generate a password hash and add it to your host_vars/master
  * `ssh root@10.8.0.1 jexec ldap slappasswd`.
  * Add the output to `slapd_root` entry in host_vars/master.
  * Reconfigure ldap jail: `ansible-playbook -i hosts site.yml --tags ldap`.

## Adding a new jail
* Edit `host_vars/master` to add the jail name to the jails list and and an IP entry.
* Edit `roles/jails/templates/network.j2` to add a new IP address to the bridge for the jail.
* Edit `roles/jails/templates/jail.conf.j2` and add a new jail entry.
* Edit `roles/bind/templates/dns.local.j2` to add a forward DNS entry for the jail.
* Edit `roles/bind/templates/reversedns.local.j2` to add a reverse DNS entry for the jail.
* If necessary, edit `roles/pf/templates/pf.conf.j2` for appropriate firewall rules.
* Make sure the new role for the jail installs a resolv.conf that points to local DNS:
  * `nameserver {{ jail_bridge_subnet }}.{{ dns_ip }}`
  * `search local`
* Run playbook
* On master, run:
  * `export http_proxy="http://squid:3128"`
  * `export JAIL=jailname`
  * `freebsd-update -b /jailhome/$JAIL fetch`
  * `freebsd-update -b /jailhome/$JAIL install`
  * `pkg -j $JAIL update`
  * `jail -r $JAIL ; jail -cmr $JAIL`
