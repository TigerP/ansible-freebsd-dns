freebsd-dns
===========

[![Build Status](https://travis-ci.org/vbotka/ansible-freebsd-dns.svg?branch=master)](https://travis-ci.org/vbotka/ansible-freebsd-dns)

[Ansible role.](https://galaxy.ansible.com/vbotka/freebsd-dns/) Configure DNS in FreeBSD.


Requirements
------------

No requiremenst.


Variables
---------

TBD (Check the defaults).


Workflow
--------

1) Change shell to /bin/sh.

```
ansible do-bsd-test -e 'ansible_shell_type=csh ansible_shell_executable=/bin/csh' -a 'sudo pw usermod freebsd -s /bin/sh'
```

2) Install role.

```
ansible-galaxy install vbotka.freebsd-dns
```

3) Fit variables. Zones can be configured when DNSSEC keys are available. *dnssec-keygen* binary is needed to generate the keys.

```
~/.ansible/roles/vbotka.freebsd-dns/vars/main.yml
```

4) Create and run the playbook.

```
> cat ~/.ansible/playbooks/freebsd-dns.yml
---
- hosts: do-bsd-tetst
  become: yes
  become_method: sudo
  roles:
    - role: vbotka.ansible-freebsd-dns
    
> ansible-playbook ~/.ansible/playbooks/freebsd-dns.yml
```

5) Create keys as described in [Authoritative DNS Server Configuration](http://www.freebsd.org/doc/en_US.ISO8859-1/books/handbook/network-dns.html#dns-dnssec-auth).

Example:

```
> cd /usr/local/etc/namedb/keys
> dnssec-keygen -f KSK -a RSASHA256 -b 2048 -n ZONE example.com
> ln -s Kexample.com.+008+20191.key Kexample.com.KSK.key
> ln -s Kexample.com.+008+20191.private Kexample.com.KSK.private
> dnssec-keygen -a RSASHA256 -b 2048 -n ZONE example.com
> ln -s Kexample.com.+008+35529.key Kexample.com.ZSK.key
> ln -s Kexample.com.+008+35529.private Kexample.com.ZSK.private
> chown bind K*
```  

6) Configure the zones.

Example of bsd_named_conf_zone:

```
bsd_named_conf_zone:
- zone: "example.com"
type: "master"
reverse: "yes"
zone_ip: "10.1.0.10"
zone_in: "0.1.10"
primary: "ns.example.com"
primary_ip: "192.168.1.10"
admin: "admin.example.com"
serial: "2016102201"
refresh: "10800"
retry: "3600"
expire: "604800"
negative: "300"
server:
- "ns1.example.com"
- "ns2.example.com"
mx:
- { server: "mx1.example.com", priority: "10" }
- { server: "mx2.example.com", priority: "20" }
host:
- { host: "srv", ip: "10.1.0.10" }
- { host: "ns1", ip: "10.1.0.2" }
- { host: "ms2", ip: "10.1.0.3" }
- { host: "mx1", ip: "10.1.0.4" }
- { host: "mx2", ip: "10.1.0.5" }
alias:
- "www"
- "mail"
```

7) Run the playbook

```
ansible-playbook ~/.ansible/playbooks/freebsd-dns.yml
```

8) Sign the zones, reload the server and test the server as described in [Authoritative DNS Server Configuration](http://www.freebsd.org/doc/en_US.ISO8859-1/books/handbook/network-dns.html#dns-dnssec-auth). The zones can be signed when the DNSSEC keys are included in the zone files.

Sign the zone. Change to the *keys* directory. Otherwise full path to the keys is needed.

```
> cd /usr/local/etc/namedb/keys
> dnssec-signzone -o example.com -k Kexample.com.KSK /usr/local/etc/namedb/master/example.com  Kexample.com.ZSK.key
> /usr/local/etc/rc.d/named reload
```

Test the server.

```
> dig @resolver +dnssec se ds 
```

9) Update registrar DS records.

- [Add a DS record](https://uk.godaddy.com/help/add-a-ds-record-23865)
- [Domain Name System Security (DNSSEC) Algorithm Numbers](http://www.iana.org/assignments/dns-sec-alg-numbers/dns-sec-alg-numbers.xhtml)
- [DS records at Godaddy with .co tld](https://lists.opendnssec.org/pipermail/opendnssec-user/2015-April/003288.html)
- [How To Setup DNSSEC on an Authoritative BIND DNS Server](https://www.digitalocean.com/community/tutorials/how-to-setup-dnssec-on-an-authoritative-bind-dns-server--2)

10) Consider to test the server with

- [dnscheck.iis.se](http://dnscheck.iis.se/)
- [DNS Check in Pingdom Tools](http://dnscheck.pingdom.com/)
- [Pingdom. Failed to deliver email. Can safely ignore this failure.](http://serverfault.com/questions/748923/how-can-i-fix-these-soa-dns-problems)
- [Veisign LABS](http://dnssec-debugger.verisignlabs.com/)
- [DNS VIZ](http://dnsviz.net/)


TODO
----
- automate creation of the keys
- automate signing of the zones
- automate testing of the server


References
----------

- [DNSSEC in 6 minutes](http://static.usenix.org/event/lisa08/dnssec_bof.pdf)
- [Domain Name System (DNS)](https://www.freebsd.org/doc/en_US.ISO8859-1/books/handbook/network-dns.html)
- [Authoritative DNS Server Configuration](http://www.freebsd.org/doc/en_US.ISO8859-1/books/handbook/network-dns.html#dns-dnssec-auth)
- [ISC: BIND 9 Configuration Reference](https://ftp.isc.org/isc/bind9/cur/9.10/doc/arm/Bv9ARM.ch06.html)
- [ISC: Inline Signing in ISC BIND 9.9.0](https://kb.isc.org/article/AA-00626/0/Inline-Signing-in-ISC-BIND-9.9.0-Examples.html)
- [ISC: BIND DNSSEC Guide](https://users.isc.org/~jreed/dnssec-guide/dnssec-guide.html)
- [Zytrax: DNS for Rocket Scientists](http://www.zytrax.com/books/dns/)
- [Zytrax: BIND (Berkeley Internet Name Domain)](http://www.zytrax.com/books/dns/ch5/)
- [Zytrax: DNS BIND Zone Transfers and Updates](http://www.zytrax.com/books/dns/ch7/xfer.html)
- [Enabling DNSSec in Bind](http://networking.ringofsaturn.com/Unix/dnssec.php)
- [Invalid TLD error when changing nameservers in godaddy](https://www.howtoforge.com/community/threads/invalid-tld-error-when-changing-nameservers-in-godaddy.62932/)
- [www.bind9.net](http://www.bind9.net/)
- [www.dnssec.net](http://www.dnssec.net/)


License
-------

[![license](https://img.shields.io/badge/license-BSD-red.svg)](https://www.freebsd.org/doc/en/articles/bsdl-gpl/article.html)


Author Information
------------------

[Vladimir Botka](https://botka.link)
