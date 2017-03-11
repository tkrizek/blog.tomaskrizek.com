---
layout: post
title: Code signing in git
tags:
  - pgp
  - git
seealso:
  - link: /2017/03/03/introduction-to-pgp
    label: Introduction to PGP
update: 2017-03-12
---

Once you [create a PGP keypair](/2017/03/04/getting-started-with-gpg2/), you can use it to sign your code in git.

## Configuring git 

First, you need to configure git to use `gpg2` for signing.

```console
$ git config --global gpg.program gpg2
```

You can also specify which PGP key should be used for signing in this particular repository.

```console
$ git config user.signingkey $KEY_ID
```

## Signing code

You can manually sign a commit or tag with the `-S` flag. For example:

```console
$ git commit -S
$ git tag -S
$ git commit --amend -S
$ git rebase -S
```

However, it's much more convenient to configure git to use PGP signing by default.

```console
$ git config --global commit.gpgsign true
$ git config --global tag.gpgsign true
```

### GPG agent

If you're signing code often, you can [configure GPG agent](/2017/03/04/getting-started-with-gpg2/#gpg-agent) to avoid re-entering your passphrase all the time.

## Viewing signatures

Signatures in git are not displayed by default. You can manually view and verify code signatures using the `--show-signature` flag.

```console
$ git log --show-signature
$ git show --show-signature
```

Signed commits will have gpg information attached.

```text
commit 3053f3735a54337da858b519b7f2a05ec9d44c35
gpg: Signature made Sat 11 Mar 2017 10:29:37 AM CET
gpg:                using RSA key 0xFA4A7FE8A9586F79
gpg: Good signature from "Tomas Krizek <tomas.krizek@mailbox.org>" [ultimate]
Author: Tomas Krizek <tomas.krizek@mailbox.org>
```

If you're viewing signatures often, you might want to set up an alias.

```console
$ git config --global alias.l "log --show-signature"
```

## Uploading you public key to GitHub

If you upload your public key to GitHub, you'll get `Verified` flag next to your signed commits and tags in GitHub's web interface.

You need to get your public key in ASCII-armored format.

```console
$ gpg2 --armor --export $KEY_ID
```

Go to your GitHub account settings and add you public key in the `SSH and GPG keys`  menu.

## Code signing in community projects

When you submit a pull request or a patch to a community project, it's up to the project maintainers how to handle your signature. If they care about code signing, they have a couple of options.

1. A commit can be accepted exactly as is, including your signature.
2. One of the project's maintainers signs the commit instead.

The second approach is the only option if the maintainers decide to modify your commit in any way (rebase, change the commit message, ...). By signing the commit themselves, they certify that the code has gone through the approval process the project has in place (code reviews, sanity checks, ...).
