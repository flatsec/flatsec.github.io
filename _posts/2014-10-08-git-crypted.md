---
layout:     post
title:      Git Crypted
date:       2014-10-08 12:31:19
summary:    Use git-crypt to easily encrypt sensitive information in git repositories.
categories:
  - encryption
  - git

---

It's pretty easy to accidentally commit password files into git (believe me I know, because I've done it). It's possible to [remove a bad commit](https://help.github.com/articles/remove-sensitive-data/), but once that information leaks out, it has to be considered compromised.

<!-- more -->

[A few months ago](http://www.webmonkey.com/2013/01/users-scramble-as-github-search-exposes-passwords-security-details/):

>[github] temporarily shut down some parts of the site-wide search update it launched...the new search tools made it much easier to find passwords, private ssh keys and security tokens stored in GitHub repos.

The quote sort-of makes it sound like github did something to expose users sensitive information, but that's not the case. Rather the issue was that there was so much sensitive information already committed that when github improved their search capability it just became much easier to find it.

To avoid this situation typically sensitive files are either not stored in the repository, or encrypted before they are pushed up. In this post I'll talk about git-crypt which is one way to accomplish encrypting files in a git repository.

### Configuration management tools

I am a big fan of configuration management tools such as Ansible, Chef, and Pupppet. Because these tools allow the expression of infrastructure as code, their configuration and/or source files can be placed in version control, and this usually includes sensitive information like passwords (example: setting the mysql servers root password).

Each of these systems has their own way of encrypting sensitive files.

* Ansible has [vault](http://docs.ansible.com/playbooks_vault.html)
* Chef has [encrypted data bags](https://docs.getchef.com/essentials_data_bags.html#encrypt-a-data-bag-item)
* Puppet has [hiera-gpg](https://github.com/crayfishx/hiera-gpg)
* and there are likely more...

Each of the above has its own workflow, but usually there is a shared key of some kind and it is used to decrypt the files at runtime. That shared key has to be readable by the process running the configuration management system.

### git-crypt

However, what I want to discuss in this blog post is [git-crypt](https://github.com/AGWA/git-crypt) which can work in any situation in which git is being used, so git-crypt could be considered, in some ways, an alternative to systems like Ansible Vault, though they are not exactly the same, and may not fit into your particular workflow or security requirements.

### Installing git-crypt

That is the easy part, at least if you are on OSX and are using brew.
On OSX using brew:

```bash
curtis$ brew install git-crypt
```

There are [install instructions](https://github.com/AGWA/git-crypt/blob/master/INSTALL.md) available for other operating sytems.

### Using git-crypt

First, I should note that git-crypt supports using gpg. gpg is probably the way to go here, but gpg can be complicated to use, so for the purposes of this blog post I'm just going to use the "shared key" workflow. So the key that is used to encrypt files in the repo must be securely shared between whoever needs to decrypt the files. Again, if you can use gpg I'd suggest doing so.

I've created a [quick example repository](https://github.com/curtisgithub/git-crypt-example) on github. There is just two files in it right now, a readme file and a passwords file. (If you click on the github link and go to the passwords file you can see it is stored as data on the github server, not plain text.)

``` bash
curtis$ tree
.
├── README.md
└── secrets
    └── passwords

1 directory, 2 files
```

I also have a ```.gitattributes``` file that specifies which files to encrypt using git-crypt.

```bash
curtis$ cat .gitattributes
secrets/passwords filter=git-crypt diff=git-crypt
```

I've been keeping my git-crypt keys in ```~/.git-crypt``` but they can be anywhere.

```bash
curtis$ git crypt keygen ~/.git-crypt/example
curtis$ file ~/.git-crypt/example
/Users/curtis/.git-crypt/example: data
```

I've already initialized this directory as a git repository.

At this point we can initialize with git-crypt.

```bash
curtis$ git crypt init ~/.git-crypt/example
```

While I have the repo unlocked and am working locally, the ```secrets/passwords``` file is in plain text. (Obviously these passwords are just examples and are not actually used anywhere.)

```bash
curtis$ cat secrets/passwords
mysql_root_password: ohr4aed8OoxoS6d
root_password: uRohmai5gieng5j
```

Once I push the repo up to github, the file will be encrypted and will not be stored in github in plain text.

If I clone the repo in another directory and cat the passwords file, we can see it just comes up as data.

```bash
curtis$ git clone git@github.com:curtisgithub/git-crypt-example.git git-crypt-example-encrypted
Cloning into 'git-crypt-example-encrypted'...
remote: Counting objects: 6, done.
remote: Compressing objects: 100% (5/5), done.
remote: Total 6 (delta 0), reused 6 (delta 0)
Receiving objects: 100% (6/6), done.
Checking connectivity... done.
curtis$ cat git-crypt-example-encrypted/secrets/passwords
GITCRYPT???9??????5??f?ۮ??D%??&6.E??|??0F:X??!?c?R?,?"???ہ ??Gǝ???curtis$
```

If I run git-crypt with init and the location of the shared key file, we can decrypt the passwords file.

```bash
curtis$ cd git-crypt-example-encrypted
curtis$ git-crypt init ~/.git-crypt/example
curtis$ cat secrets/passwords
mysql_root_password: ohr4aed8OoxoS6d
root_password: uRohmai5gieng5j
```

If I change or add passwords and such to the passwords file, commit it, and push it up to the central github server, the file will be re-encrypted.

### Conclusion

I like [git-crypt](https://github.com/AGWA/git-crypt) because it's pretty simple to use. I think information technology is battling with complexity right now, so simple is good. It's also nice to be able to store passwords and other sensitive information right in the repository with the rest of the code. Using gpg with git-crypt would also make for a pretty good workflow.

There is a lot more to think about in terms of keeping secrets, but for the purposes of this blog post I just wanted to go over a quick introduction to git-crypt using a shared key file, which might be a good way to get your team introduced to encrypting files in git repository.
