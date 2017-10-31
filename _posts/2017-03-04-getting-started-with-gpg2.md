---
layout: post
title: Getting started with gpg2
tags:
  - pgp
seealso:
  - link: /2017/03/11/code-signing-in-git/
    label: "Code signing in git"
  - link: /2017/10/31/pgp-key-signing/
    label: "PGP Key Signing"
update: 2017-03-12
---

This article describes how to configure your environment, generate a [PGP](/2017/03/03/introduction-to-pgp/) keypair and backup your private key.

## Prerequisites

```console
# dnf install gnupg2 pinentry-gnome3
```

`gpg2` program is used in the following commands. There's a chance you will also have `gpg` installed. *Make sure you're using the same version for all commands. This is a common source of mistakes.*

There are multiple versions of `pinentry-*`. This package provides a basic dialog for entering the passphrase. Install the one that suits your needs.

## Configuration

There are two important configuration files.

* `~/.gnupg/gpg.conf` is the main GPG configuration file
* `~/.gnupg/gpg-agent.conf` configures the GPG agent (see below)

### Key ID format

When working with PGP keys, there are three basic identifiers you can see:

* `fingerprint` which uniquely identifies the keypair
* `0xlong` key ID -- last 16 digits of the fingerprint
* `0xshort` key ID -- last 8 digits of the fingerprint

```text
fingerprint   2BB553947645C5829937FB63B2F27B7221DCD363
0xlong                              0xB2F27B7221DCD363
0xshort                                     0x21DCD363
```

`0xshort` key ID format is used by default. I recommend to use the more accurate `0xlong` format. You can set it in `gpg.conf`.

```text
keyid-format 0xlong
```

### <a name="gpg-agent"></a> GPG agent

If you don't want to enter your passphrase every time you send an e-mail or create a commit, you can set up a GPG agent.

First, enable the GPG agent in `gpg.conf`.

```text
use-agent
```

### TTL cache

It's up to you to decide how long you want to keep the password cached.

For example, if you always lock your computer when you leave it and want to enter your password just once a day, you could choose to keep it cached for 10 hours.

You can configure the TTL in `gpg-agent.conf`. The time is specified in seconds.

```text
default-cache-ttl 36000
max-cache-ttl 36000
```

## Creating your keypair

You can use the following command to create your keypair.

```console
$ gpg2 --full-gen-key
Please select what kind of key you want:
   (1) RSA and RSA (default)
   (2) DSA and Elgamal
   (3) DSA (sign only)
   (4) RSA (sign only)
Your selection? 1
```

Choose the first option to generate RSA key for both signing and encryption.

### Keysize

```console
RSA keys may be between 1024 and 4096 bits long.
What keysize do you want? (2048) 4096
```

* `4096` is recommended for new keys
* `2048` should be secure for the next decade
* `1024` *should not be considered secure*

### Expiration date

```console
Please specify how long the key should be valid.
         0 = key does not expire
      <n>  = key expires in n days
      <n>w = key expires in n weeks
      <n>m = key expires in n months
      <n>y = key expires in n years
Key is valid for? (0) 1y
```

It's a very good idea to limit the validity of your keys in case you lose your private key and the revocation certificate.

### Identity

```console
Real name: First Last
Email address: first.last@example.com
Comment: 
You selected this USER-ID:
    "First Last <first.last@example.com>"
```

Fill in your `uid`. Once you generate the key, you can attach additional `uid`s to it.

### Passphrase

Make sure to select a strong, high-entropy passphrase. Something like [xkcd#936](https://www.xkcd.com/936/) style passphrase. You can use a technique such as [diceware](http://world.std.com/~reinhold/diceware.html).

*Do not forget your passphrase. It is not possible to recover it and you can't use your keypair without it.*

## Revocation certificate

A revocation certificate is automatically generated in `~/.gnupg/openpgp-revocs.d/$FINGERPRINT.rev`. You can use this to revoke your keypair in case your private key is compromised or lost.

## Backup

It's a good idea to back up your private key to a secure location. You can export it in ASCI armored format with the following command.

```console
$ gpg2 --armor --export-secret-keys first.last@example.com > private.key.asc
```

You usually don't need to back up your public key, since you can usally recover it from other sources (e.g. keyservers). If you want to backup your public key as well, you can do it with the following command.

```console
$ gpg2 --armor --export first.last@example.com > public.key.asc
```

For long-term backup/storage, consider using a tool like [paperkey](http://www.jabberwocky.com/software/paperkey/) to make a paper backup. When your private key is backed up, it's still protected by your passphrase.
