---
layout: post
title: Introduction to PGP
tags:
  - pgp
seealso:
  - link: /2017/03/04/getting-started-with-gpg2/
    label: "Getting started with gpg2"
update: 2017-03-07
---

[OpenPGP](https://en.wikipedia.org/wiki/Pretty_Good_Privacy#OpenPGP) uses public key cryptography to provide features such as digital signatures and message encryption. The standard doesn't rely on any single certificate authority. Instead, its users build a web of trust among themselves.

## Terms

* `OpenPGP` commonly used standard defined in [RFC4880](https://tools.ietf.org/html/rfc4880) 
* `PGP` (Pretty Good Privacy) as originally invented by Phil Zimmermann
* `GPG` (GNU Privacy Guard) an implementation of OpenPGP standard

## Why Use PGP

### E-mail signatures

It is a trivial task to forge e-mail headers, which include the sender's address. Anyone can send an e-mail that seemingly comes from you. When you sign an e-mail with PGP, the e-mail becomes *temper-proof*, *authentic* and *indesputable*.

### Source code signatures

In open-source communities, it is a common practice to sign the software's source code or packages. Once signed, anyone can verify the source code wasn't tempered with since it has been signed by the developers or packagers.

### E-mail encryption

When you need to transmit sensitive data (access codes, passwords, confidental documents, ...) to another person over the internet, you can use PGP to encrypt the message for one particular recipient.

### File encryption

Symmetric key encryption (like [AES](https://en.wikipedia.org/wiki/Advanced_Encryption_Standard)) is usually used to encrypt files, because it demands less hardware resources. However, PGP can also be used to encrypt files.

## How it works

PGP relies on [public key cryptography](https://en.wikipedia.org/wiki/Public-key_cryptography) -- you need a keypair, a public and private key. The public key is meant to be distributed to others through **keyservers** or other means. The private key should be kept secure and only used by its owner. *Protecting the private key is crucial, since anyone who has the private key can impersonate you.*

### Keyservers

Keyservers, such as [pgp.mit.edu](https://pgp.mit.edu), make it easier to handle public keys. Major keyserver usually synchronize with each other.

Anyone can upload a public key to the keyserver. Also, anyone can issue a keypair with a chosen identity. Since there is no central authority, it is up to every user to decide which key to trust.

### Who to trust

There are many reasons why you decide to trust a key and they depend on your particular use case.

* Personally verifying someone's identity and their key
* Getting the key from a project's / developer's website
* Relying on the **web of trust**
* Trust the key on first use

### Key signing

If you decide to trust another key and verify that it belongs to the owner, you can certify the key with your signature. Afterwards, you can publish this signature to contribute to the **web of trust**.

### Web of trust

> As time goes on, you will accumulate keys from other people that you may want to designate as trusted introducers. Everyone else will each choose their own trusted introducers. And everyone will gradually accumulate and distribute with their key a collection of certifying signatures from other people, with the expectation that anyone receiving it will trust at least one or two of the signatures. This will cause the emergence of a decentralized fault-tolerant web of confidence for all public keys.

Phil Zimmermann, according to [Wikipedia](https://en.wikipedia.org/wiki/Web_of_trust)
