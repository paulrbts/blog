---
layout: post
title: Setting up Postfix for sending mail
meta: There are lots of guides on setting up Postfix but I found few of them worked with Mac and also GMX as a provider. Here's a summary of how to do it.
category: software
tags: [email, mac]
---
<h3 class="page.title">
  {% if page.title %}
    <a href="{{ site.baseurl }}{{ page.url }}">{{ page.title }}</a>
  {% endif %}
</h3>

**{{ page.date | date_to_long_string }}**

___
#### Credit to others
The work in here was really done by other people.
In particular, one guide that really helped me was from [Ben von der Weiden](https://bitbucket.org/benjamin-von-der-weiden/smartmontools-mac-osx-howto/src) with his guide on setting up smartmontools.
During this guide, Ben talked about how to set up Postfix so that you can receive external mails with updates on errors.

For the most part I'm just going to quote Ben, as he's already said what needs to be said.
I've edited some steps in the process, though.

#### Setup
The very first thing to do is sign up for a free email account just to be used for this computer.
I'm going to refer to this email address as server-email@provider.com throughout.
If I refer to your-email@address.com then that's, well, your normal email address.

One final point, because there's a mixture of text to input into files and command line input, you should be aware that where the usual shell commands (e.g. sudo vi /etc/file) you don't put that into the file but you enter it to the shell after adding the previous command.
Having said that, I've tried to split the file input away from the shell commands.

Open your shell programme, then we'll move to the directory and copy the original file, so that we have our vanilla settings available:

```sh
cd /etc/postfix
sudo cp -i -p main.cf main.cf.default.orig
sudo vi /etc/postfix/main.cf
```

First check out the IMAP settings for your host then add the following lines below the commented out relayhosts, replacing the part in square brackets if it's different for your provider:

```sh
relayhost = [smtp.provider.com]:587
smtp_sasl_auth_enable = yes
smtp_sasl_password_maps = hash:/etc/postfix/sasl_passwd
smtp_sasl_security_options = noanonymous
smtp_use_tls = yes
```

Find the section below about address rewriting (otherwise GMX will reject the emails as 'unauthorised sender') then add the following uncommented lines below:

```sh
# ADDRESS REWRITING
#
# The ADDRESS_REWRITING_README document gives information about
# address masquerading or other forms of address rewriting including
# username->Firstname.Lastname mapping.
sender_canonical_maps = pcre:/etc/postfix/canonical_sender
recipient_canonical_maps = pcre:/etc/postfix/canonical_recipient
```

Now we'll edit the postfix settings, some of this is on the usual Postfix setup guides, but the part about setting up the canonical_sender and recipient that we just started that I found to be really important:

```sh
sudo vi /etc/postfix/sasl_passwd
```

add the following line:

```sh
[smtp.provider.com]:587 server-email@provider.com:password
```

```sh
sudo chmod 600 /etc/postfix/sasl_passwd
sudo postmap /etc/postfix/sasl_passwd
```

Now for the canonical_sender:

```sh
sudo touch /etc/postfix/canonical_sender
sudo chmod 644 /etc/postfix/canonical_sender
sudo vi /etc/postfix/canonical_sender
```

add the following line, then we're moving on to canonical_recipient:

```sh
/.+/ server-email@provider.com
```

```sh
sudo touch /etc/postfix/canonical_recipient
sudo chmod 644 /etc/postfix/canonical_recipient
sudo nano /etc/postfix/canonical_recipient
```

add the following line:

```sh
/.+/ your-email@address.com
```

```sh
cp -i -p /etc/aliases /etc/aliases.default
sudo vi /etc/aliases
```

Edit the following line:

```sh
# Person who should get root's mail. Don't receive mail as root!
root:				your-email@address.com
```

below all specific aliases add this:
```sh
*:				your-email@address.com

sudo newaliases
```
send a test email:

```sh
date | mail -s "subject" your-email@address.com
```

#### Sources
* [Ben von der Weiden's Smartmontools how- to](https://bitbucket.org/benjamin-von-der-weiden/smartmontools-mac-osx-howto/src)
