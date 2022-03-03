# How to Sign Your git Commits and Tags ?

**Why Do We Need to Sign Our Commits?**

`git` uses `user.name` and `user.email` values in your configuration file which `~/.gitconfig` (mostly). `git` uses **3 Levels of Configuration** logic:

1. Project level: `./.git/config`: When you create a git repository, `.git/` folder is also created automatically and stores all the git related database, config files etc...
2. User level: `~/.gitconfig`: This affects user’s options. We all use this.
3. System-wide level: `/etc/gitconfig`: Used for global configuration options such as extra helpers etc...

To use `git`, all we need is a `user.name` and `user.email`. They are both strings, texts nothing fancy about it. You can define anything you like:

```
$ cd /tmp/
$ git init demo-repo && cd $_
Initialized empty Git repository in /private/tmp/demo-repo/.git/
$ git config user.name "Rob Pike"
$ git config user.email "r@golang.org"
```

Let’s check the config:

```
$ git config --local -l
core.repositoryformatversion=0
core.filemode=true
core.bare=false
core.logallrefupdates=true
core.ignorecase=true
core.precomposeunicode=true
user.name=Rob Pike
user.email=r@golang.org
```

Let’s verify;

```
$ cat .git/config
[core]
	repositoryformatversion = 0
	filemode = true
	bare = false
	logallrefupdates = true
	ignorecase = true
	precomposeunicode = true
[user]
	name = Rob Pike
	email = r@golang.org
```

As you can see, the legendary **Rob Pike** is now one of our contributors. Let’s make some commit:

```
$ git commit --allow-empty -m 'hello from rob'
$ git commit --allow-empty -m 'golang is not that good, use rust!'
```

Well, let’s see:

```
$ git log
commit bb4322e56eedf750ce1efeee03165a7bdb21a052 (HEAD -> main)
Author: Rob Pike <r@golang.org>
Date:   Mon Jan 31 16:49:38 2022 +0300

    golang is not that good, use rust!

commit 1b7ab45683ce00bbfbb7a53c565ab60ddaa227fd
Author: Rob Pike <r@golang.org>
Date:   Mon Jan 31 16:49:19 2022 +0300

    hello from rob
```

Now, let’s verify one of Rob’s commits. Since `git 2.5` we have two commands for verification;

1. `git verify-commit`
2. `git verify-tag`

```
$ git verify-commit bb4322e56eed
gpg: Signature made Mon Jan 31 16:49:38 2022 +03
gpg:                using RSA key 19CACF8F3AB406F6D6445842D0972B1BBE84BBE4
gpg: Good signature from "Ugur Ozyilmazel (MacBook Pro 15retina - Bilgi) <ugurozyilmazel@gmail.com>" [ultimate]
```

Boom! according to signature, this commit belongs to "[ugurozyilmazel@gmail.com](mailto:ugurozyilmazel@gmail.com)" If we use `--raw` option:

```
$ git verify-commit --raw bb4322e56eed
[GNUPG:] NEWSIG
[GNUPG:] KEY_CONSIDERED 19CACF8F3AB406F6D6445842D0972B1BBE84BBE4 0
[GNUPG:] SIG_ID tOogMG2apfL8V8T7ThV71H1dfhQ 2022-01-31 1643636978
[GNUPG:] KEY_CONSIDERED 19CACF8F3AB406F6D6445842D0972B1BBE84BBE4 0
[GNUPG:] GOODSIG D0972B1BBE84BBE4 Ugur Ozyilmazel (MacBook Pro 15retina - Bilgi) <ugurozyilmazel@gmail.com>
[GNUPG:] VALIDSIG 19CACF8F3AB406F6D6445842D0972B1BBE84BBE4 2022-01-31 1643636978 0 4 0 1 8 00 19CACF8F3AB406F6D6445842D0972B1BBE84BBE4
[GNUPG:] TRUST_ULTIMATE 0 pgp
```

We have another option for displaying signatures;

```
$ git log --show-signature
commit bb4322e56eedf750ce1efeee03165a7bdb21a052 (HEAD -> main)
gpg: Signature made Mon Jan 31 16:49:38 2022 +03
gpg:                using RSA key 19CACF8F3AB406F6D6445842D0972B1BBE84BBE4
gpg: Good signature from "Ugur Ozyilmazel (MacBook Pro 15retina - Bilgi) <ugurozyilmazel@gmail.com>" [ultimate]
Author: Rob Pike <r@golang.org>
Date:   Mon Jan 31 16:49:38 2022 +0300

    golang is not that good, use rust!

commit 1b7ab45683ce00bbfbb7a53c565ab60ddaa227fd
gpg: Signature made Mon Jan 31 16:49:19 2022 +03
gpg:                using RSA key 19CACF8F3AB406F6D6445842D0972B1BBE84BBE4
gpg: Good signature from "Ugur Ozyilmazel (MacBook Pro 15retina - Bilgi) <ugurozyilmazel@gmail.com>" [ultimate]
Author: Rob Pike <r@golang.org>
Date:   Mon Jan 31 16:49:19 2022 +0300

    hello from rob
```

Commits are signed yes, also indicates **Good signature** but not from Rob :)

In order to ensure the accuracy of the **commit**, we must sign it with our key/credentials. Let’s do it!

---

## **Download Required Tools and Generate Key**

If you are on a macOS or Linux, you probably have installed [Homebrew](https://brew.sh/) right? The following examples only apply to **macOS**.

```
$ brew install gnupg pinentry-mac
```

- `gnupg` package contains `gpg` related tools
- `pinentry-mac` is a macOS app for handling `gpg` password entry operations

Let’s create `gpg-agent.conf`;

```
$ gpg
```

You are now in the interactive mode, just hit ⌃ + C (control+C) and exit. This creates `~/.gnupg` folder with correct permissions. Let’s find the path of `pinentry-mac`;

```
$ command -v pinentry-mac
# /opt/homebrew/bin/pinentry-mac    # for apple m1
# /usr/local/bin/pinentry-mac       # for intel
```

Now let’s create config file;

```
$ nano ~/.gnupg/gpg-agent.conf
```

```
default-cache-ttl 600
max-cache-ttl 7200
pinentry-program /path/to/pinentry-mac

```

Now let’s generate:

```
$ gpg --full-generate-key
```

**Please select what kind of key you want:**

```
gpg (GnuPG) 2.3.4; Copyright (C) 2021 Free Software Foundation, Inc.
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.

Please select what kind of key you want:
   (1) RSA and RSA
   (2) DSA and Elgamal
   (3) DSA (sign only)
   (4) RSA (sign only)
   (9) ECC (sign and encrypt) *default*
  (10) ECC (sign only)
  (14) Existing key from card
Your selection? <HIT ENTER>

```

Number (9) is the default, let’s keep that encryption algorithm. Hit enter end continue;

**Please select which elliptic curve you want:**

```
Please select which elliptic curve you want:
   (1) Curve 25519 *default*
   (4) NIST P-384
   (6) Brainpool P-256
Your selection? <HIT ENTER>

```

Number (1) is the default, let’s keep in that way. Hit enter end continue;

**Please specify how long the key should be valid.**

```
Please specify how long the key should be valid.
         0 = key does not expire
      <n>  = key expires in n days
      <n>w = key expires in n weeks
      <n>m = key expires in n months
      <n>y = key expires in n years
Key is valid for? (0)

```

I've been using the same key for years, therefore I select `0`.

**Key does not expire at all**

```
Is this correct? (y/N) y

```

Type `y` and hit return.

**GnuPG needs to construct a user ID to identify your key.**

```
GnuPG needs to construct a user ID to identify your key.

Real name: No Turkish Chars Allowed
Email address: thisis@ademo.com
Comment: This is a demo purpose gpg key
You selected this USER-ID:
    "No Turkish Chars Allowed (This is a demo purpose gpg key) <thisis@ademo.com>"

Change (N)ame, (C)omment, (E)mail or (O)kay/(Q)uit? O

```

Enter your real name and email address. Do not use strange letters such as Turkish letters like `ğĞ`. Use ASCII only... When you fill all, Type `O` (O for Okay) and hit return.

Now, `pinentry-mac` pops up:

Enter your password then you are good to go:

```
We need to generate a lot of random bytes. It is a good idea to perform
some other action (type on the keyboard, move the mouse, utilize the
disks) during the prime generation; this gives the random number
generator a better chance to gain enough entropy.
We need to generate a lot of random bytes. It is a good idea to perform
some other action (type on the keyboard, move the mouse, utilize the
disks) during the prime generation; this gives the random number
generator a better chance to gain enough entropy.
gpg: directory '/Users/vigo/.gnupg/openpgp-revocs.d' created
gpg: revocation certificate stored as '/Users/vigo/.gnupg/openpgp-revocs.d/1257475AE097714568CBDAF894CEADAD66DD9BED.rev'
public and secret key created and signed.

pub   ed25519 2022-01-31 [SC]
      1257475AE097714568CBDAF894CEADAD66DD9BED
uid                      No Turkish Chars Allowed (This is a demo purpose gpg key) <thisis@ademo.com>
sub   cv25519 2022-01-31 [E]

```

Now we have a key! Let’s list them:

```
$ gpg -k
gpg: checking the trustdb
gpg: marginals needed: 3  completes needed: 1  trust model: pgp
gpg: depth: 0  valid:   3  signed:   0  trust: 0-, 0q, 0n, 0m, 0f, 3u
/Users/vigo/.gnupg/pubring.kbx
------------------------------
pub   ed25519 2022-01-31 [SC]
      1257475AE097714568CBDAF894CEADAD66DD9BED
uid           [ultimate] No Turkish Chars Allowed (This is a demo purpose gpg key) <thisis@ademo.com>
sub   cv25519 2022-01-31 [E]
```

In `uid` section, you see: `[ultimate]`. This means this key is trusted key!

---

## **Let’s Sign Our Commits**

Now, It’s time to sign our commits with newly generated key. `1257475AE097714568CBDAF894CEADAD66DD9BED` is our sign key. We need to add this to our git configuration:

```
$ git config user.signingKey 1257475AE097714568CBDAF894CEADAD66DD9BED # use this key
$ git config commit.gpgsign true                                      # yeah! sign them!
```

Let’s check our **local configuration**:

```
$ git config --local -l
core.repositoryformatversion=0
core.filemode=true
core.bare=false
core.logallrefupdates=true
core.ignorecase=true
core.precomposeunicode=true
user.name=Rob Pike
user.email=r@golang.org
user.signingkey=1257475AE097714568CBDAF894CEADAD66DD9BED
commit.gpgsign=true
```

We have configured out git tool to sign commits/tags via using `1257475AE097714568CBDAF894CEADAD66DD9BED` key. Don’t forget, we are working on **locally**. Nothing affects user level git config at the moment.

Now, we are ready to commit:

```
$ git commit --allow-empty -m 'my first signed commit'
```

Now, again `pinentry-mac` pops up and asks for key’s password. Write `demo124` as password:

Do not forget to check "**Save in Keychain**" option otherwise you will have to enter the password every time you commit your changes!

```
[main aeedcbd9df73] my first signed commit

$ git log --show-signature

$ git log --show-signature
commit aeedcbd9df73a402605600ab30c177d9707b4245 (HEAD -> main)
gpg: Signature made Mon Jan 31 17:37:21 2022 +03
gpg:                using EDDSA key 1257475AE097714568CBDAF894CEADAD66DD9BED
gpg: Good signature from "No Turkish Chars Allowed (This is a demo purpose gpg key) <thisis@ademo.com>" [ultimate]
Author: Rob Pike <r@golang.org>
Date:   Mon Jan 31 17:37:21 2022 +0300

    my first signed commit
:
:
:
:
```

---

## **Let’s Add Our Key to GitHub**

First, let’s export our key’s public version;

```
$ gpg --armor --export 1257475AE097714568CBDAF894CEADAD66DD9BED
-----BEGIN PGP PUBLIC KEY BLOCK-----

mDMEYffuMBYJKwYBBAHaRw8BAQdAdrt+y4fNJPEJ0ZsZKyGiXMYDR3EaLnznfuhi
3Otp5ru0TE5vIFR1cmtpc2ggQ2hhcnMgQWxsb3dlZCAoVGhpcyBpcyBhIGRlbW8g
cHVycG9zZSBncGcga2V5KSA8dGhpc2lzQGFkZW1vLmNvbT6IlAQTFgoAPBYhBBJX
R1rgl3FFaMva+JTOra1m3ZvtBQJh9+4wAhsDBQsJCAcCAyICAQYVCgkICwIEFgID
AQIeBwIXgAAKCRCUzq2tZt2b7QdhAPsHR7k2uzcLrlL5jHcVlL7mWjiYqO7fZx2E
uiSyF063ZAEAsrfy1hbVBj9hw9ORddA1qQ4DaCMGn1FFm6Rwm++w/g64OARh9+4w
EgorBgEEAZdVAQUBAQdALMWMB/XK5bPm4uK4lCyHU966HOCBHZYvNcIl8VX0bFQD
AQgHiHgEGBYKACAWIQQSV0da4JdxRWjL2viUzq2tZt2b7QUCYffuMAIbDAAKCRCU
zq2tZt2b7anlAP4qveyB9n4hvy7bwtNEjVmXCd+T0ah2O66+NsW+kj1EuQEArB1H
Rekgm4zmYnm+OgkZTJMwsXNwlR42iLA4w6VU9A8=
=dx2S
-----END PGP PUBLIC KEY BLOCK-----
```

You can copy to clipboard via;

```
$ gpg --armor --export 1257475AE097714568CBDAF894CEADAD66DD9BED | pbcopy
```

Now open GitHub, go to your settings [https://github.com/settings/keys](https://github.com/settings/keys); Click "New GPG key":

Now, paste **Key** input area;

Now, hit "Add GPG Key", enter your GitHub account credentials (*password*), now you are good to go:

If you verify that email from [https://github.com/settings/emails](https://github.com/settings/emails) page via **Add email address** form, you will not see that **unverified** indicator.

---

## **Backup or Transfer Your Keys**

When you move to new computer, you need to transfer you keys. You can backup all or some of your keys;

```
$ gpg --export-secret-key -a > /path/to/keys/all-keys.asc     # export all of the keys
$ gpg --export-secret-key KEY > /path/to/keys/some-key.asc    # export only given KEY
```

You can import your keys via;

```
$ gpg --import /path/to/keys/all-keys.asc
$ gpg --import /path/to/keys/some-key.asc
```

---

## **Trust after Import**

When you complete import operation, you need to **trust** your key again!

```
$ gpg --edit-key 1257475AE097714568CBDAF894CEADAD66DD9BED

gpg> trust
:
:
  1 = I don't know or won't say
  2 = I do NOT trust
  3 = I trust marginally
  4 = I trust fully
  5 = I trust ultimately     # <-
  m = back to the main menu

Your decision? 5
```

Type `5`, trust ultimately! and hit enter.;

```
Do you really want to set this key to ultimate trust? (y/N) y
:
:[ultimate] (1). No Turkish Chars Allowed (This is a demo purpose gpg key) <thisis@ademo.com>

```

Make sure you see `[ultimate]` text!. Now hit `quit` and exit. In the interactive shell, try `help` to see other features.

---

## **Sign Your Tags**

All you need is to add `-s`:

```
$ git tag -s v0.0.0 -m 'my signed tag for version 0.0.0'
$ git tag -v v0.0.0 # verify
object aeedcbd9df73a402605600ab30c177d9707b4245
type commit
tag v0.0.0
tagger Rob Pike <r@golang.org> 1643642050 +0300

my signed tag for version 0.0.0
gpg: Signature made Mon Jan 31 18:14:10 2022 +03
gpg:                using EDDSA key 1257475AE097714568CBDAF894CEADAD66DD9BED
gpg: Good signature from "No Turkish Chars Allowed (This is a demo purpose gpg key) <thisis@ademo.com>" [ultimate]

$ git verify-tag v0.0.0 # shorter
gpg: Signature made Mon Jan 31 18:14:10 2022 +03
gpg:                using EDDSA key 1257475AE097714568CBDAF894CEADAD66DD9BED
gpg: Good signature from "No Turkish Chars Allowed (This is a demo purpose gpg key) <thisis@ademo.com>" [ultimate]
```

To sign them automatically,

```
$ git config tag.gpgSign true
$ git config --local -l
core.repositoryformatversion=0
core.filemode=true
core.bare=false
core.logallrefupdates=true
core.ignorecase=true
core.precomposeunicode=true
user.name=Rob Pike
user.email=r@golang.org
user.signingkey=1257475AE097714568CBDAF894CEADAD66DD9BED
commit.gpgsign=true
tag.gpgsign=true
```

To apply commit and tag signing globally (*user level*) add `--global` to config set command;

```
$ git config --global commit.gpgSign true
$ git config --global tag.gpgSign true
```

You can always verify via checking your `~/.gitconfig` file;

```
$ cat ~/.gitconfig # or
$ cat ~/.gitconfig | grep gpgSign
    gpgSign = true
    gpgSign = true
```

---

## **Delete Key**

Delete operations are always dangerous, please check twice before you delete anything!

```
$ gpg --delete-secret-keys KEY # delete private key
$ gpg --delete-keys KEY        # delete public key

# completely remove...
$ gpg --delete-secret-keys 1257475AE097714568CBDAF894CEADAD66DD9BED
$ gpg --delete-keys 1257475AE097714568CBDAF894CEADAD66DD9BED
```

---

## **GPG Failure**

Sometimes commit signing fails due to macOS shell environment. To fix this; we need to kill gpg agent;

```
$ gpgconf --kill gpg-agent
$ echo "testing gpg" | gpg --clearsign
-----BEGIN PGP SIGNED MESSAGE-----
Hash: SHA256

testing gpg
-----BEGIN PGP SIGNATURE-----

iQEzBAEBCAAdFiEEGcrPjzq0BvbWRFhC0JcrG76Eu+QFAmH6GxgACgkQ0JcrG76E
u+RDyQf8DOxsU72tB+r1XTkClzAx4Nr6mbmy5e9DCkzjxogH5Ip69VVRTIEShSmT
1ycajSWO8dh+SXBZbSrrQVmCeS2a7neYYZJ4wbK2sxGF70sbFNwGQA6eVntRET+z
VrZ1WonJbjBRTYFgDJIrnmfvzfbo8SNyipUM4GpbohPNL1RmHPkWqB71y4OABaH7
D3Ph7zIUpKqTmc/oWmZo0pQeGGtjq50uVya/pT6Zf8OwDPX1GhACBHxndiDFB2lx
bOud+bYc1gmwdxCW8NW2vafd+J8R9ZzdjKAudS96hTzedL7OlA6/SSMGrxt+s9VE
mdYA0zYJr39tEkgwU+oW+q7GPDryNQ==
=BbP5
-----END PGP SIGNATURE-----
```

This means, gpg is back again! If not, you should try to add;

```
export GPG_TTY=$(tty)
```

to your bash profile or bashrc.

---

## **Resources**

- [https://docs.github.com/en/authentication/managing-commit-signature-verification](https://docs.github.com/en/authentication/managing-commit-signature-verification)
- [https://www.gnupg.org/gph/en/manual/book1.html](https://www.gnupg.org/gph/en/manual/book1.html)
- [https://yanhan.github.io/posts/2017-09-27-how-to-use-gpg-to-encrypt-stuff/](https://yanhan.github.io/posts/2017-09-27-how-to-use-gpg-to-encrypt-stuff/)
- [https://github.com/yemeksepeti/ys-how-to-guides](https://github.com/yemeksepeti/ys-how-to-guides)