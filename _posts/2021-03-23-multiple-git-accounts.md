---
title: Working with multiple GitHub accounts
layout: single
toc: true
tags: git github ssh shell development multiple accounts switching
---

My old GitHub account was full of snippets, dead projects and stuff that makes sense only to me but still I don't want to get rid of. So when I started this site I opted for creating a new account to host the source and my more recent projects.

Hence the need to manage two different GitHub accounts. In this post I will describe what you need in order to manage two or more GitHub accounts with different SSH keys.


## Requisites
- Basic GNU/Linux console knowledge
- Two or more GitLab accounts 
- A working ssh client configuration

If you work on Linux box ssh is very likely to be properly installed. This guide deals only with SSH access to GitLab since the SSH protocol is the more convenient and practical way to connect with GitHub without supplying username and personal access token over and over.

## GitHub SSH access overview
GitHub allows you to authenticate with SSH keys, but id doesn't create a new SSH user for each account. Everyone is authenticating with the _git_ SSH user and the system recognises the GitHub account by the SSH key. 

You can generate a new SSH key by issueing:
```bash
$ ssh-keygen -t rsa -b 2048 -C "your_email@example.com"
```

You will be prompted to enter a passphrease for the newly created key, you can leave it blank if you don't want a passphrase:
```bash
sdurz@linuxbox:~$ ssh-keygen 
Generating public/private rsa key pair.
Enter file in which to save the key (/home/sdurz/.ssh/id_rsa): 
```


then, follow [GitHub instructions](https://docs.github.com/en/github-ae@latest/github/authenticating-to-github/adding-a-new-ssh-key-to-your-github-account) to add your key to your existing account.


### GitHub SSH hosts
When you issue a command like:

```bash
$ git co git@gitlab.com:/auser/repo.git
```

you are basically specifiying a ssh _host_ and path to the SSH client.. The SSH client will look for a suitable authentication method for the host and it will 


an _host_ can be replaced at any time with a corresponding _Host_ entry as 
defined in a SSH configuration file (namely, the _~/.ssh/config_ file in your home directory).

Say you have the following declarations in _~/.ssh/config_:
```
Host github
Hostname github.com
User git
IdentityFile /home/sandro/.ssh/id_rsa
```

then you could issue the same command as above by typing:
```bash
$ git co github:/auser/repo.git
```

## Multiple aliases

By configuring different aliases for github.com you can select the GitHub account you are willing to use by connecting with the corresponding SSH _Host_.

So I renamed the _Host_ definition to _github-old_:
```
# Old account dual75
Host github-old
Hostname github.com
User git
IdentityFile /home/sandro/.sdurz/id_rsa
```

Then I added a second definition:
```
# New account sdurz
Host github
Hostname github.com
User git
IdentityFile /home/sandro/.ssh/id_rsa_new
```

Now I have two _Host_ definition for GitHub, they differ only in the SSH key. The old one still uses my original key, the newer uses my new key.

### Testing your SSH aliases


```bash
$ ssh -T github-old
Hi dual75! You've successfully authenticated, but GitHub does not provide shell access.

$ ssh -T github
Hi sdurz! You've successfully authenticated, but GitHub does not provide shell access.
```

That's it.

From now on I will switch between my older or newer GitHub or by using _github-old_ or _github_.
















## Further steps

