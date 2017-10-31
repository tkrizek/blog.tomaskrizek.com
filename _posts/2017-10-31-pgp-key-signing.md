---
layout: post
title: PGP Key Signing
tags:
  - pgp
update: 2017-10-31
---

Since there's no central certrificate authority in PGP, users often sign each
others' keys to build a decentralized web of trust. This usually involves
exchaning fingerprints of keys, verifying each others' identities and e-mail
addresses and signing each others' keys afterwards.

## Obtaining the key

To sign someone's key, you have to import it to your local keyring. Users
usually upload their keys to a keyserver to be able to conveniently exchange
them. To obtain a key from a keyserver, issue the following command.

```console
$ gpg2 --recv-keys $KEY_ID
```

Note that there are multiple popular keyservers and this contacts just one of
them (as defined by your configuration). However, the major keyservers are
synchronized with each other, so the key should be avaiable after some period
of time, even if it was uploaded to a different keyserver.

## Signing the key

You have the option to use trust signatures if you want to [delegate
trust](https://www.linuxfoundation.org/blog/pgp-web-of-trust-delegated-trust-and-keyservers/).
In this post, I'm using regular signatures and the classic trust model, in
which you only trust the keys you've signed yourself.

After you've verified the fingerprint of the key, you can signed the associated
`uid`s. You should only sign the `uid`s you've verified.

```console
$ gpg2 --edit-key $KEY_ID
gpg> sign
gpg> save
```

Afterwards, you can check the signatures on a key.

```console
$ gpg2 --list-sigs $KEY_ID
```

## Publishing the signed key

Once you've signed someone's key, you want to deliver the signed key back to
them. You have the option to do it properly, by sending them the signed key in
an encrypted e-mail sent to the e-mail address in the `uid` you've signed. This
way you've verified the associated e-mail address is valid and has access to
the private key, because they can't publish and use your signature without access
to the private key.

If you've verified their e-mail address in some other way, you can also just
upload the signed key to the keyserver and they can download it afterwards.

```console
$ gpg2 --send-keys $KEY_ID
```
