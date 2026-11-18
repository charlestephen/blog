---
title: "Automating TLS Over a DNSSEC-Signed Cluster (Because I Can't Leave Well Enough Alone)"
date: 2026-11-10T09:00:00-05:00
draft: false
tags: ["cloudflare", "dns", "dnssec", "acme", "tls", "technitium", "homelab"]
categories: ["networking"]
summary: "Why I built ACME-over-RFC-2136 against a signed DNS cluster instead of just pointing certbot at Cloudflare's API like a normal person."
cover:
  image: ""
  alt: ""
  caption: ""
---

Most people doing ACME DNS-01 at home point `certbot` or `acme.sh` at Cloudflare's API, get a token, and move on with their lives. That works. I did not do that, because my internal DNS is a DNSSEC-signed, multi-node Technitium cluster, and plain API-token ACME doesn't have an opinion about any of that — it just wants a TXT record to appear somewhere. I wanted the TXT record to appear *correctly*, on a cluster that signs its own zones, without hand-editing anything every 90 days.

## The actual problem

DNS-01 needs a way to write a `_acme-challenge` TXT record into a zone, prove it, and clean it up after. The easy version of that is "call a DNS provider's REST API." The harder version — the one you get when the zone in question is self-hosted and signed — is RFC 2136 dynamic updates: TSIG-authenticated, pushed straight to the authoritative server, no API layer in between.

RFC 2136 against a DNSSEC zone is where this stops being a weekend script. Every dynamic update has to land correctly across the cluster, get re-signed, and propagate before the ACME client checks for it — and if your zone replication or signing is even slightly out of sync, the challenge fails in a way that looks like a DNS bug, not a cert bug, and you spend an hour debugging the wrong layer. Technitium handles this natively once it's configured for it; the dynamic-update path and the signing path aren't fighting each other, they're the same zone.

I'll say up front: this exact path barely gets exercised. I don't issue internal, hostname-based certs very often — almost everything on the internal side is a cert for a bare IP, which doesn't need DNS-01 at all (there's no hostname to prove ownership of). But the capability's there and wired correctly for the rare case, which matters more to me than whether it runs daily.

## Where the certificate manager sits

I didn't want every service on the network running its own ACME client against the DNS cluster directly — that's N things holding real credentials instead of one. So there's a certificate manager (UCM) sitting in the middle, and it's the *only* thing with actual API access to anything. Nothing downstream ever sees a Cloudflare token or a TSIG key.

For publicly trusted certs, the path is simple and never touches the internal DNS cluster at all: UCM talks to Cloudflare's API to drop the `_acme-challenge` TXT record, Let's Encrypt validates it, UCM gets back the cert and private key. Technitium has no role in anything that ends up signed by Let's Encrypt — that separation is deliberate.

The part that actually saves work: UCM holds onto what it issues. A client asking for a cert UCM already has on file gets handed that cert instead of triggering a fresh issuance — so a wildcard cert, once issued, gets reused across every consumer that wants it instead of re-running the ACME flow per client.

## The other half: UCM as its own CA

For anything that isn't a publicly-resolvable name, UCM stops being an ACME *client* and becomes the CA itself — it runs its own internal ACME endpoint, with itself as the signing authority, for hosts that have no business touching Cloudflare or Let's Encrypt. In practice this is almost entirely bare-IP certs, which sidestep the DNS-01 question entirely since there's no name to validate.

On the rare occasion it *is* a hostname rather than an IP, the flow comes back to the DNS cluster: the client's RFC 2136 update lands on UCM's own listener first, and UCM relays it into Technitium rather than Technitium fielding it directly — one theoretical chokepoint for every write to the zone, whether the request originated internally or is headed out to Cloudflare. (In practice, all of UCM's data exists in a replicated Postgres database with a hot spare UCM version on a different server waiting to take over at the edit of two DNS records.)

Trust for the internal CA isn't credential-based by default, but it's not purely automatic either — it's a choice I make per client. Most clients are configured to request a cert with no credential at all, and get one just by being reachable from UCM at a LAN address. Where a client supports real enrollment, I configure it to use a shared generic credential or a per-client EAB key instead — not because reachability-based trust is insufficient, but because an actual enrolled identity lets me tell what a given device on the LAN actually is, instead of just "something answered at this IP."

That's the actual value of the layer: it turns "90-day cert expiry" from a calendar reminder into a thing that simply doesn't happen anymore, on infrastructure where DNSSEC makes the DNS side of that automation non-trivial to get right — even for the path I rarely use.

## Why I'm writing this down

Short version: because I run everything else this way too.

I don't do this because a client is paying me to. Nobody is grading my homelab. I built an authoritative DNSSEC cluster with catalog-based zone replication, wired ACME into it over RFC 2136 instead of the easy API path, and automated the whole renewal loop — for a network where the only stakeholder is me and Home Assistant and some smart tech that isn't so smart. That's the tell. If I hold my own infrastructure to a standard most people reserve for production-with-an-SLA, that's not a personality quirk, it's just how I build things, and it doesn't change when there's a client or an employer attached to the outcome instead of just my own curiosity.

If you're evaluating whether I'm the kind of engineer who cuts corners when no one's watching: this is the answer. Nobody's watching here. I still signed the zones. I still am running my own CA. I still automated it all.
