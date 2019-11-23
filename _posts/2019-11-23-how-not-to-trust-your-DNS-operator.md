---
layout: post
title: How not to trust your DNS operator
tags:
  - dns
update: 2019-11-23
---

The DNS privacy debate usually centers around encryption, whether it's
DNS-over-TLS or DNS-over-HTTPS (which may actually do [more harm than
good](https://labs.ripe.net/Members/bert_hubert/centralised-doh-is-bad-for-privacy-in-2019-and-beyond)).
However, encryption itself only protects the data in transit. The operator of
the DNS resolver still sees all the queries and may (ab)use them for their own
interests or be liable to share them with other parties.

The only protection of the users' queries at the resolver is policy - whether
self-proclaimed Privacy Policy, legally binding GDPR, or simply the operator's
policy of not logging the users' queries at all. And while there might be
better or worse DNS resolver operators, ultimately, the user is at their mercy.

In the finance world, investors often
[diversify](https://en.wikipedia.org/wiki/Diversification_(finance)) their
portfolio. Instead of putting all their money in a single company and risk
losing everything, the investment is split among multiple companies. In case
any of the companies fail, some value is lost, but no single failure is fatal.

I think the same can be done when it comes to picking your DNS resolver.  Why
go for just a single operator and risks exposing *all* of your traffic in case
they have malicious intent? Isn't it better to use multiple operators to
resolve *different* domains, so if any of the resolvers you picked are
malicious, they don't have your complete DNS profile.

A naive way to implement this would be to send your DNS queries to a randomly
selected resolver.  Beware of this approach, as it's more of a snakeoil rather
than a real solution. It is no good when `tomaskrizek.com A` is sent to one
resolver and `blog.tomaskrizek.com A` (or `tomaskrizek.com AAAA`) to another --
in fact, it's worse than before, since you're now exposing your browsing habits
to multiple resolvers. The same applies when resolving the same query just
moments later, but sending it to a different resolver.

But despair not, Knot Resolver comes to the rescue! It supports [forwarding to
multiple
resolvers](https://knot-resolver.readthedocs.io/en/stable/modules.html#forwarding-to-multiple-targets)
while ensuring each resolver is used to resolve a different set of domains. For
details, see
[policy.slice_randomize_psl()](https://knot-resolver.readthedocs.io/en/stable/modules.html#c.policy.slice_randomize_psl).

I'm the author of this functionality and I have to admit, I expected to run
into some serious issues when using it. I imagined nightmarish debugging of
cases where only certain domains fail to resolve, while others work. I imagined
endlessly messing with my config file, trying to figure out which one of the
resolvers I forward my queries to have gone dark...

I've been using this feature a couple of months now. I've been splitting my
entire DNS traffic into 6 distinct subsets and using up to 12 different
resolvers simultaneously to resolve them. I didn't notice any strange behaviour
or outages. I didn't have to meddle with my config and go through any of the
nightmarish debugging I imagined. In other words -- my DNS just works, as it
should.

The most important part is using at least two independent resolvers for every
single slice and choosing reliable and trustworthy resolver operators.  I
picked the resolvers I liked from
[dnsprivacy.org](https://dnsprivacy.org/wiki/display/DP/DNS+Privacy+Test+Servers)
and this is the configuration snippet I've been using for my local Knot
Resolver:

```lua
-- apply any policies you want *before* using policy.slice()
-- this one turns off default DNS-over-HTTPS in Firefox!
policy.add(policy.suffix(policy.DENY, {todname('use-application-dns.net')}))

-- TLS-forward 6 distinct slices up to 12 resolvers (with both IPv4 and IPv6)
policy.add(policy.slice(
	policy.slice_randomize_psl(),
	policy.TLS_FORWARD({
		{'199.58.81.218', hostname='dns.cmrg.net'},
		{'2001:470:1c:76d::53', hostname='dns.cmrg.net'},
		{'159.69.198.101', hostname='dot-de.blahdns.com'},
		{'2a01:4f8:1c1c:6b4b::1', hostname='dot-de.blahdns.com'},
	}),
	policy.TLS_FORWARD({
		{'146.185.167.43', hostname='dot.securedns.eu'},
		{'2a03:b0c0:0:1010::e9a:3001', hostname='dot.securedns.eu'},
		{'89.234.186.112', hostname='dns.neutopia.org'},
		{'2a00:5884:8209::2', hostname='dns.neutopia.org'},
	}),
	policy.TLS_FORWARD({
		{'94.130.110.185', hostname='ns1.dnsprivacy.at'},
		{'2a01:4f8:c0c:3c03::2', hostname='ns1.dnsprivacy.at'},
		{'94.130.110.178', hostname='ns2.dnsprivacy.at'},
		{'2a01:4f8:c0c:3bfc::2', hostname='ns2.dnsprivacy.at'},
	}),
	policy.TLS_FORWARD({
		{'145.100.185.18', hostname='dnsovertls3.sinodun.com'},
		{'2001:610:1:40ba:145:100:185:18', hostname='dnsovertls3.sinodun.com'},
		{'145.100.185.17', hostname='dnsovertls2.sinodun.com'},
		{'2001:610:1:40ba:145:100:185:17', hostname='dnsovertls2.sinodun.com'},
	}),
	policy.TLS_FORWARD({
		{'37.252.185.232', hostname='dot1.appliedprivacy.net'},
		{'2a00:63c1:a:229::3', hostname='dot1.appliedprivacy.net'},
		{'89.233.43.71', hostname='unicast.censurfridns.dk'},
		{'2a01:3a0:53:53::0', hostname='unicast.censurfridns.dk'},
	}),
	policy.TLS_FORWARD({
		{'116.203.70.156', hostname='dot1.dnswarden.com'},
		{'2a01:4f8:1c1c:75b4::1', hostname='dot1.dnswarden.com'},
		{'116.203.35.255', hostname='dot2.dnswarden.com'},
		{'2a01:4f8:1c1c:5e77::1', hostname='dot2.dnswarden.com'},
	})
))
```

Note that in some cases, the impact on privacy might be negative. Specifically,
if a website uses multiple registrable domains to deliver content (e.g.
`twitter.com` and `twimg.com`), these queries might be exposoed to different
resolvers.

However, I believe the overall impact on privacy is positive. It decentralizes
your DNS and reduces the trust you have to put in any single operator. Combined
with [DNS-over-TLS](/2019/06/22/dns-privacy-with-dot/) that protects the
queries in transit, I think this is best DNS privacy you can get short of using
Tor.
