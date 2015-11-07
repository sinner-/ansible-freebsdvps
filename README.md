# ansible-freebsdvps
Building on the work from https://github.com/sinner-/freebsdfun
this is an ansible project to manage a VM based FreeBSD VPS. 

## Goals
* TODO: Manage PF based firewall.
* Manage OpenVPN based VPN.
** TODO: Provide VPN gateway functionality.
* TODO: Manage jails and jail networking.
** TODO: squid web proxy jail.
** TODO: BIND DNS server jail.
** TODO: Default jail configuration to use squid and BIND.
** TODO: kerberos auth jail.
** TODO: LDAP directory jail.
*** TODO: LDAP to use kerberos.
** TODO: postfix MTA jail.
*** TODO: postfix to use kerberos+LDAP.
** TODO: apache httpd jail.
* TODO: Manage LDAP entries and kerberos accounts for users.

## Before running
* before running, need to generate OpenSSL certificates and copy to:
** roles/openvpn/templates/cacert.pem
** roles/openvpn/templates/openvpncert.pem
** roles/openvpn/templates/openvpnkey.pem
** roles/openvpn/templates/openvpndh4096.pem
** roles/openvpn/templates/tls-auth.pem

## To run
* run ansible-playbook -i hosts site.yaml
