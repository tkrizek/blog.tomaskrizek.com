---
layout: post
title: DNS Privacy with DoT
tags:
  - dns
  - dot
update: 2019-06-22
---

Almost every internet communication begins with a DNS query. These are commonly
transmitted over UDP in plain text. DNS over TLS (DoT, [RFC
7858](https://tools.ietf.org/rfc/rfc7858.txt)) uses a secure channel to
communicate with the DNS server. This has two major benefits. First, it hides
your queries from 3rd parties (local network operator, ISP, ...) during
transit. Second, it ensures you receive the query as it was sent from the DNS
server and no one has manipulated it in transit (popular with ISPs and port
53/udp).

If you want to take advantage of DoT, one of the options you have is to use a
local resolver and forward your queries over TLS to another resolver. However,
this comes with another issue - who are you forwarding your queries to? What
will they do with your queries? Does their bussiness model include data mining
your private data? Be sure to check out the privacy policies of those big DoT
providers.

In the spirit of decentralization, I'd like to encourage you to consider using
a privacy-oriented resolver that doesn't log your queries or IP address, such
as [dns.dns-tls.com](https://github.com/tomaskrizek/resolver-dns-tls-deploy)
(operated by me), some of those listed at
[dnsprivacy.org](https://dnsprivacy.org/wiki/display/DP/DNS+Privacy+Test+Servers)
or run by nonprofit organizations, such as [CZ.NIC's ODVR](https://www.nic.cz/odvr/).

It's important to note DoT only protects your DNS queries. There are other
methods to monitor your traffic. One example would be
[SNI](https://en.wikipedia.org/wiki/Server_Name_Indication), which sends a
domain name in plaintext during TLS handshake. To mitigate these, I'd recommend
to use Tor and/or VPN.
