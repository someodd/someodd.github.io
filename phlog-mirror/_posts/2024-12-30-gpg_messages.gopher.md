---
date: 2024-12-30
tags:
- security
title: Using GPG to encrypt messages (like email)

---
Original content in gopherspace: gopher://gopher.someodd.zip:Just 7071/phlog/


I wanted to email phrack...

I downloaded their public key as `phrack.asc`

https://phrack.org/contact

This is for staff(at)phrack[DOT]org

## encrypting a message for them

then i imported the public key: `gpg --import phrack.asc`

then make a message like `email.txt` and then:

```
gpg --encrypt --armor -r staff@phrack.org email.txt
```

then copy its contents and paste into the body of email (or attach it). i just pasted into claws mails as plaintext

don't forget to sign it! signing it is just letting them know it actually came from you, they'll be able to view it without knowing your public key. like this:

```
gpg --encrypt --sign --armor --local-user someodd@pm.me -r staff@phrack.org email.txt
```

## keep your key updated!

if you need to update your key (you should keep it updated!) make a message that's signed.

first create the new key with `gpg --full-generate-key` or whatever. then write a message like this `key-update.txt`:

```
I, someodd@pm.me, am transitioning to a new PGP key. My new key fingerprint is:

[NEW_KEY_FINGERPRINT]

This message is signed with my old PGP key to prove my identity.
```

sign the statement with this:

```
gpg --sign --armor --local-user someidhere key-update.txt
```

This creates a signed file, key-update.txt.asc, that anyone can verify using your old public key. then others can:

```
gpg --verify key-update.txt.asc
gpg --import new-public-key.asc
```

Don't forget to revoke the old key if not used:

```
gpg --output revoke.asc --gen-revoke <OldKeyID>
```

I'm not going to get into keyservers.

Set key expiration.


