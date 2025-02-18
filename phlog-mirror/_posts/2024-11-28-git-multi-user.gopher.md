---
date: 2024-11-28
tags:
- development
- linux
- git
- security
title: git, GitHub and multiple accounts/profiles

---


## SSH Key

Create the key, be sure to use a name like `id_ed25519_username`, it's also recommended to set a passphrase for added security:

```
ssh-keygen -t ed25519 -C "you@example.org"
```

Add the key to your SSH agent:

```
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519_username
```

### GitHub, Repo

Add the key to your GitHub account:

```
cat ~/.ssh/id_ed25519_username.pub
```

Copy the output to whatever the "new ssh key" dialog is in your GitHub profile.

Configure SSH for specific GitHub repos (you'll need to change the repos use
this) in `~/.ssh/config`:

```
Host github.com-username
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_ed25519_username
```

You can update the remote URL for a repo like this:

```
git remote set-url origin git@github.com-username:ghusername/repo.git
```

### GPG Key

Generate the GPG key:

```
gpg --full-generate-key
```

Choose RSA with at least 4096 bits. Set expiration date (or leave it as 0 for
no expiration). Enter name and email when prompted.

Before we go further, you may want to backup your public and private keys.
First get the ID for your key.

```
gpg --list-secret-keys --keyid-format=long
```

Then you can do something like this:

```
gpg --armor --export somekeyhere > username_public_key.asc
gpg --armor --export-secret-keys somekeyhere > username_private_key.asc
```

Moving on, add the GPG key to GitHub (should be similar in the GitHub interface
to adding a new SSH key):

```
gpg --armor --export somekeyhere
```

For these remaining configurations, I think there's a better way to do this,
but configure `git` to use the GPG key for a specific repo:

```
git config user.signingkey somekeyhere
git config commit.gpgsign true
```

Also, config `git` to use a specific username and email for this repo:

```
git config user.name "full name"
git config user.email "you@example.org"
```

You can make signed commits like this:

```
git commit -S -m "Test signed commit"
```

## Git configuration per domain

This is a more maintainable approach to have defaults set per domain we have
(matching our `~/.ssh/config` domains, used in repos):

Create a `~/.gitconfig-username`:

```
[user]
    name = Full Name
    email = user@example.org
    signingkey = somekeyhere
[commit]
    gpgsign = true
```

Update global git configuration (`~/.gitconfig`):

```
[includeIf "hasconfig:remote.*.url:git@github.com-username:*/**"]
    path = ~/.gitconfig-username
```

Check the applied settings in a repo that uses the domain `github.com-username`:

```
git config --get user.name
git config --get user.email
git config --get user.signingkey
git config --get commit.gpgsign
```

Original content in gopherspace: gopher://gopher.someodd.zip:7071/phlog/
