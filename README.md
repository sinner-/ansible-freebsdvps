# ansible-freebsdvps
Building on the work from https://github.com/sinner-/freebsdfun
this is an ansible project to manage a VM based FreeBSD VPS. 

## Goals
* Manage PF based firewall.
* Manage OpenVPN based VPN.
  * TODO: Provide VPN gateway functionality.
* TODO: Manage server software:
  * TODO: squid web proxy.
  * BIND DNS.
  * kerberos auth.
  * TODO: LDAP directory.
  * TODO: LDAP to use kerberos.
  * TODO: postfix MTA.
    * TODO: postfix to use kerberos+LDAP.
  * TODO: apache httpd.
* TODO: Manage LDAP entries and kerberos accounts for users.

## Before running
* before running, need to generate OpenSSL certificates and copy to:
  * roles/openvpn/templates/certificates/cacert.pem
  * roles/openvpn/templates/certificates/openvpncert.pem
  * roles/openvpn/templates/certificates/openvpnkey.pem
  * roles/openvpn/templates/certificates/openvpndh4096.pem
  * roles/openvpn/templates/certificates/tls-auth.pem

## To run
* boot your VPS, preferably with the label "master".
* copy hosts.example to hosts and update the IP address of master.
* run `ansible-playbook -i hosts site.yml`.
* after the first run is complete, update your hosts file to point to the VPN address of the server.
  * `sed -i 's/1.1.1.1/master.local' hosts`
