---
layout: post
title: Introduction to PGP
tags:
  - pgp
seealso:
  - link: index.md
    label: "[TODO] Getting Started with PGP"
update: 2017-03-04
---

OpenPGP uses public key cryptography to provide features such as digital signatures and message encryption.
The standard doesn't rely on any single certificate authority. Instead, its users build a web of trust among themselves.

## Terms

* `OpenPGP` commonly used standard defined in [RFC4880](https://tools.ietf.org/html/rfc4880) 
* `PGP` (Pretty Good Privacy) as originally invented by Phil Zimmermann
* `GPG` (GNU Privacy Guard) an implementation of OpenPGP standard

## Why Use PGP

### E-mail Signatures

It is a trivial task to forge e-mail headers, which include the sender's address. Anyone can send an e-mail that seemingly comes from you. When you sign an e-mail with PGP, the e-mail becomes *temper-proof*, *authentic* and *indesputable*.

### Source Code Signatures

In open-source communities, it is a common practice to sign the software's source code or packages. Once signed, anyone can verify the source code wasn't tempered with since it has been signed by the developers or packagers.

### E-mail Encryption

When you need to transmit sensitive data (access codes, passwords, confidental documents, ...) to another person over the internet, you can use PGP to encrypt the message for one particular recipient.

### File Encryption

Symmetric key encryption (like AES) is usually used to encrypt files, because it demands less hardware resources. However, PGP can also be used to encrypt files.

## How It Works

PGP relies on [public key cryptography](https://en.wikipedia.org/wiki/Public-key_cryptography) - you need a key pair, a public and private key. The public key is meant to be distributed to others through **keyservers** or other means. The private key should be kept secure and only used by its owner. *Protecting the private key is crucial, since anyone who has the private key can impersonate you.*

### Keyservers

Keyservers, such as [pgp.mit.edu](https://pgp.mit.edu), make it easier to handle public keys. Anyone can upload a public key to the keyserver. Also, anyone can issue a key pair with a chosen identity. Since there is no central authority, it is up to every user to decide which key to trust.

### Who To Trust

There are many reasons why you decide to trust a key and they depend on your particular use case.

* Personally verifying someone's identity and their key
* Getting the key from a project's / developer's website
* Relying on the **web of trust**
* Trust the key on first use

### Key Signing

When you decide to trust another key, you certify it with your signature. Afterwards, you can publish this signature to contribute to the **web of trust**.

### Web of Trust

You've already decided to trust Alice's, Bob's and Carol's key. You've met them yourself and verified their identities. One day, you get a signed message from Dan. You've never met him before and have no reason to trust he is who he claims he is. However, his key is signed by Alice, Bob and Carol. By extension, you can choose to trust Dan, even though you've never met him.

