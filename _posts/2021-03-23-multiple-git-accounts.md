---
title: Working with multiple GitHub accounts
layout: single
toc: true
tags: git github ssh shell development multiple accounts switching
category: development
---

My old GitHub account was a complete mess. Code snippets, configuration files, dead projects and crappy stuff that made sense only to me, I didn't want to get rid of the whole thing but when I started this site I opted for creating a new account. The newly created account will host this site source and my recent projects.

Hence the need to manage two different GitHub accounts. In this post I will describe what you need to do in order to manage two GitHub accounts in the console.


## Requisites

- Basic GNU/Linux console knowledge
- Two or more GitLab accounts 
- A working SSH client configuration

This guide deals only with SSH access to GitLab since the SSH protocol is the more convenient and practical way to connect with GitHub without supplying username and personal access token over and over.
If you work on a Linux box SSH is very likely to be already installed and working properly. 

## GitHub SSH setup
GitHub allows you to authenticate with SSH keys, but it doesn't actually create a new SSH user for each account. On GitHub everyone is authenticating with the _git_ SSH user and the system recognises the GitHub account by the SSH key in use. Basically, if you want to access distinct account via SSH you will need distinct SSH keys.

If you don't have already a SSH key for your new account you will need to generate it and upload it.
As usual you can generate a new SSH key with _ssh-keygen_:
```bash
$ ssh-keygen -t rsa -b 2048
```

You will be prompted to enter a passphrease for the newly created key, you can leave it blank if will:
```bash
$ ssh-keygen 
Generating public/private rsa key pair.
Enter file in which to save the key (/home/sdurz/.ssh/id_rsa): 
```

Finally, follow [GitHub instructions](https://docs.github.com/en/github-ae@latest/github/authenticating-to-github/adding-a-new-ssh-key-to-your-github-account) to add your key to your new account.


### GitHub SSH hosts
When you clone a GitHUB repository, like this:

```bash
$ git clone git@github.com:auser/repo.git
```

you are basically specifiying SSH user and hostname in the form _user@hostname_ to the SSH client. Under the hood the SSH client will look through its configuration files and resolve the actual hostname and connection options before establishing the actual connection. The _hostname_ part is treated as a plain hostname only if SSH can't find a corresponding _Host_ entry in its configuration files (namely, the _~/.ssh/config_ file in your home directory or the global _/etc/ssh/config_).

Say you have the following declarations in _~/.ssh/config_:
```
Host gh
Hostname github.com
User git
```

SSH will look for a _Host_ entry matching the *gh* shortcut and determine the correct user and hostname. So you could issue the former command as:
```bash
$ git co gh:/auser/repo.git
```
Of course there is not much of use to in a shorter alias. But with proper _Host_ directives we can configure different aliases with different _IdentityFile_, different
_IdentityFile(s)_ mean different SSH keys an thus different GiHub accounts.


## Multiple _Host_ definitions for GitHub

By configuring different aliases for github.com you can select a different account just by specifying the corresponding _Host_ alias on the command line.

So I created a specific _Host_ definition, pointing to my old key:
```
# Old account dual75
Host github-old
Hostname github.com
User git
IdentityFile /home/sdurz/.sdurz/id_rsa
```

Then I added a second definition:
```
# New account sdurz
Host github
Hostname github.com
User git
IdentityFile /home/sdurz/.ssh/id_rsa_new
```

Now I have two _Host_ definitions for GitHub, they differ only in the SSH key. The old one still uses my original key, the newer uses my new key. Please note that the 
account name does not appear anywhere in the file.

### Testing the SSH aliases

You can't test the configuration by connecting via SSH without allocating a terminal (GitHub won't let you do it anyway, -T will prevent the client from trying). Here I will check for both my _Host_ definitions. 
Verify that the resulting message contains your username.


I can check my old account, _dual75_:

```bash
$ ssh -T github-old
Hi dual75! You've successfully authenticated, but GitHub does not provide shell access.
```

And the new one, _sdurz_:
```
$ ssh -T github
Hi sdurz! You've successfully authenticated, but GitHub does not provide shell access.
```

Everything seem to work properly. 

If something shouln't work check [Error: Permission denied (publickey)](https://docs.github.com/en/enterprise-server@2.20/articles/error-permission-denied-publickey)

## Ending

That's it. From now on I will switch GitHub account or just by using _github-old_ instead of _github_.

