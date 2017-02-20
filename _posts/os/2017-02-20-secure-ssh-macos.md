---
layout: post
title: SSH keys on macOS
meta: Generating and using SSH keys in a macOS client/server environment for fast and secure login.
category: os
tags: [mac, security, admin]
---
<h3 class="page.title">
  {% if page.title %}
    <a href="{{ site.baseurl }}{{ page.url }}">{{ page.title }}</a>
  {% endif %}
</h3>

**{{ page.date | date_to_long_string }}**

___
#### *Another* SSH keys guide?
Many of the guides I have read on this subject seem to have some shortcoming or other to give me the confidence for my needs: they cover Linux; they don't explain enough about different parts, *etc.*
As with other elements of this blog, this also acts as a reminder for me in the future if I'm doing it again, so I don't look at $search.engine and find I'm not happy with the results.

The whole point of SSH keys is to make it quicker to log in to remote machines.
The danger of anything which makes processes quicker (convenience) is that they usually compromise security.
In this case, we're actually increasing it.
That is, as long as your local machine is [securely setup in the first place](https://github.com/drduh/macOS-Security-and-Privacy-Guide).
The keys are going to use 2048-bit encryption, which is the default encryption level for keygens on macOS.
Before we start, logging in will be something like this:

```zsh
ssh user@remote_server.com
password: ********
```
When we're done, it'll be more like this:
```zsh
ssh lemmein
```
and it'll actually be stronger than before, because it won't rely on -- neigh, be able to be brute-forced by -- a password.
Not only is it quicker because it doesn't rely on the time you take to type in the password, but I find that the delay actually prompting for a password, followed by the delay of verifying it, was quite tiresome on occasions (up to a few seconds).
With SSH keys, login is almost instantaneous.

It uses three programmes/processes:
* `ssh-keygen` to create secure keys
* a local config file to create a shortcut to the server
* a config file on the server to limit access to authorized keys only

##### Local machine
On your local machine, your daily driver -- the one from which you want to access remote machines -- fire up Terminal or iTerm2 (we're staying here for the rest of the guide) and check whether you have keys generated already:

```zsh
ls ~/.ssh
```
If you have a pair of files in there called `id_rsa` and `id_rsa.pub` then you have keys.
You shouldn't have one and not the other.
Assuming you have neither, enter the following:

```zsh
ssh-keygen -t rsa
```
When prompted, just hit `enter` instead of using a passphrase.
This generates your `id_rsa` (private, not be shared with anyone) and `id_rsa.pub` (public, to be shared with your servers) keys.
Now, copy the public key onto the server using the secure copy programme:

```zsh
scp ~/.ssh/id_rsa.pub user@remote_server.com:.ssh/authorized_keys
```
This says: securely copy my public key to `authorized_keys` folder on the server, owned by that user.
This means you'll be able to log in as that user, and that user only, so make sure you substitute the required user on your server in the above command.

Make sure you're in your home folder and check your permissions are correct:

```zsh
cd ~
ls -al
```
This spits out permissions of all directories and files in your home folder, look for .ssh, which should read: `drwx------`
If it doesn't, type: `chmod 700 ~/.ssh` which modifies the permissions to such.

Move into that directory and check the permissions of its contents:

```zsh
cd .ssh
ls -al
```
These should be even less permissive: `-rw-------`.
If they aren't: `chmod 600 ~/.ssh`.
These could both be checked one after the other---or not checked at all, as long as you run the commands---then the commands chained together: `chmod 700 ~/.ssh && chmod 600 ~/.ssh*`

##### Server-side
Jump onto the server, make sure they key works and that the permissions are correct.
This time, you'll be looking at a directory called `authorized_keys` as well as `.ssh` (notice there is no password prompt this time):

```zsh
ssh user@remote_server.com
cd ~
ls -al
cd .ssh
ls -al
```
The permissions should be:
* `.ssh drwx------`
* `authorized_keys -rw-------`

If not, or if you can't be bothered to check: `chmod 700 ~/.ssh && chmod 600 ~/.ssh/authorized_keys`

#### Shortcut
To make all this a little quicker, we can make a shortcut using an SSH config file.
macOS has two: a system-wide config at `/etc/ssh` and a user-specific file at ~/.ssh
We'll do all this back on the daily driver.

Firstly, check to see if you have one already with `cat ~/.ssh/config`.
If you get nothing, we're good to go.
Otherwise, check it, familiarise yourself with these servers you're set up for, and maybe you've already done this bit?

```zsh
cd ~/.ssh
nano config

Host [host shortname e.g. lemmein]
  HostName remote_server.com
  User [name of user you want to log in as]
  IdentityFile=~/.ssh/id_rsa
```

One final test: `ssh lemmein`
And you should be in.

##### Preventing password access
macOS uses some OpenBSD goodness.
For example, the system-wide SSH config file I referred to earlier and an sshd config file, both residing in `/etc/ssh`.
We're going to jump onto our server and edit the sshd config file to make sure that passwords aren't accepted, only RSA authentication is a permitted way of getting in through the SSH port.

```zsh
ssh lemmein
cd /etc/ssh
nano sshd_config
```
See, it says right at the top that it's from OpenBSD.

These lines should be present but commented out (with a #).
They may have a 'yes' there, so just make sure each line matches the following, i.e. delete the # if there's one there:

```zsh
RSAAuthentication yes
PubkeyAuthentication yes
ChallengeResponseAuthentication no
PasswordAuthentication no
UsePAM no
```

Finally, just restart the SSH daemon:
```zsh
sudo launchctl stop com.openssh.sshd
sudo launchctl start com.openssh.sshd
```

##### Sources
* [Nixcraft](https://www.cyberciti.biz/faq/create-ssh-config-file-on-linux-unix/)
* [MediaTemple](https://mediatemple.net/community/products/dv/204644740/using-ssh-keys-on-your-server)
* [Matt Stauffer](https://mattstauffer.co/blog/setting-up-a-new-os-x-development-machine-part-3-dotfiles-rc-files-and-ssh-config#ssh)
