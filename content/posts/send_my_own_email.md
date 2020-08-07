---
title: Send My Own Emails
slug: send_my_own_emails
date: 2020-08-06
tags:
- email
description: Sending your own email by yourself is suprisingly hard.
---

I've been running my own email server for a while now. I have my own domain, my own email addresses, my own email server, so if you send me an email it would go through the least possible number of hops into my brain. However, up until now, I send my emails through a 3rd party vendor, [SendGrid](https://sendgrid.com/), for the very simple reason which is to be able to reach my recipients. One very prominient email provider that start with the letter G caused the most number of troubles. With sendgrid doing the deliveries, all the troubles are theirs.

## Why switch ##

My email sending needs is modest. The free plan from sendgrid is enough for me, and I could afford the lowest paid plan if I have to. However, it still feel **wrong** to use sendgrid, other than being cheap:

 * My email server is perfectly capable of sending my own emails to the final destinations. Why should my emails take a detour through a vendor, who can read or even alter my emails at least in theory?
 * I am not among the targeted customers for Sendgrid (or Mailgun, MailChimp, etc). They boast about email campaigns and news letters for companies, while I am just a guy sending emails for myself, my friends, and my family.
 * Sendgrid has certain "features" that make me unconfortable. For example, it can turn the links in my emails into [spying links](https://www.campaignmonitor.com/resources/knowledge-base/what-is-click-through-rate-how-can-ctr-be-calculated/), or insert the infamous [spying pixel](https://en.wikipedia.org/wiki/Web_beacon). Yes I opted out all of the above; However, I feel morally wrong by just being their customer, even on a free plan. 

## The pain of sending emails directly ##

Sending emails is supposed to be easy: The email server just looks up the MX record, figured out which server to contact then just shot the email over. However, my previous tries of sending emails directly have resulted in a lot of bounce emails, or emails landing in the "junk" folder. Why? And why I can't go through by myself while Sendgrid can? Yesterday I sat down and determined to solve the mysteries. Here are what I have figured out.

### IPv6 ###

First thing is easy: Disable IPv6 in your email server software [exim](https://exim.org/). I know IPv6 is supposed to solve the Internet problem and everyone should support it. However, if an email server does not have a IPv4 address the chance it can receive emails is very slim. So IPv6 does not give you any benefit while giving you twice the debugging headache. In my case, merely having IPv6 on will cause troubles talking to some hosts, thanks to my registar [domain.com](https://www.domain.com/)'s lack of proper [AAAA DNS records](https://support.dnsimple.com/articles/aaaa-record/).

### Basic DNS and rDNS ###

I had them all along, but here I want to reiterate the bassic DNS setup because of their importance. You should have the following at the minimum:

 * An A record for your domain.
 * An A record for the email server of your domain.
 * A MX record for your domain that point to your email server's domain name
 * rDNS records from both IPv4 and IPv6 addresses of your email server that point to your email server's domain name
 
### Web server and SSL ###

You should have a web server on your email server, even if you don't need an web server. Just make one serving nothing but a blank page. And that web server should have proper SSL certificate (not self-signed) so https works. [Certbot](https://certbot.eff.org/) will help you achieve this.

Your email sever should also have TLS (not self-signed) on SMTP. The certificate you get from the web server on the same machine comes in handy now; you can use the same one. You also want to add a deploy hook on cerbot to reload your email server when a new certificate is deployed. 

### SPF ###

You must have [Sender Policy Framework](https://en.wikipedia.org/wiki/Sender_Policy_Framework). It basically publishes an allow-list that tells the world which IP can send your emails through DNS. For simplicity's sake just set it to: `v=spf1 a mx ~all`. It lists 2 IPs, one from the A record of your domain, the other from the A record that is referenced by the MX record of your domain, then disable all the rest. All emails that originate from your domain must come from either one of them.

### DKIM ###

[DKIM](https://en.wikipedia.org/wiki/DomainKeys_Identified_Mail) is another way to qualify who can send your email. It feels redundant now that you have SPF, but you must have both of them. It publishes a public key through DNS, so receiving email servers can verify the signature that your email server embedded in the header of outgoing emails. The setup is a little complicated so I document in detail, using Exim4 on Debian. If you want to follow please use your own domain name instead of `3qin.us`:

#### Make the key pair ####

```
cd /etc/exim4
openssl genrsa -out "3qin.us-dkim-private.pem" 2048
openssl rsa -in "3qin.us-dkim-private.pem" -out "3qin.us-dkim-public.pem" -pubout
chmod g+r "3qin.us-dkim-private.pem"
chgrp Debian-exim 3qin.us-dkim-private.pem
```

#### Install the private key in Exim4 ####

Append the following lines to your exim4.conf.localmacros:

```
# DKIM
DKIM_CANON = relaxed
DKIM_SELECTOR = 20200803
DKIM_DOMAIN = MAIN_LOCAL_DOMAINS
DKIM_PRIVATE_KEY = /etc/exim4/MAIN_LOCAL_DOMAINS-dkim-private.pem
```

Remember, the conf file must end with a newline. I've been burned.

#### Install the public key in DNS ####

In your DNS you should have a TXT entry named `20200803._domainkey`, which is the same string as your DKIM_SELECTOR as above, appended with `._domainkey`. The value should be `k=rsa; VERYLONGUGLYPUBKEY` where the VERYLONGUGLYPUBKEY is your public key in one non-breaking string. Make sure you copied and paste the whole key without any space or carriage return in the middle!

### DMARC ###

[DMARC](https://en.wikipedia.org/wiki/DMARC) is designed to tell email domain owners what to do if either SPF or DKIM check failed. Logically you'd think there should be a sane default, but no, you have to have the rules spelled out. For simplicity's sake just add one TXT entry named `_dmarc` with value like:

```
v=DMARC1; p=quarantine; sp=reject; rua=mailto:postmaster@3qin.us
```

It tells domain owners to quarantine (put in junk folder) everything that failed either SPF or DKIM from your domain (you could have mis-configured something), and bounce everything that failed either SPF or DKIM from a sub domain of your domain (I don't have such things, so they must be bogus). It also tells the domains to send a periodic summary report to the postmaster email of your domain. (Yeah, I am really going to read that.)

That's all, just wait until your DNS records to propagate, then test. Most likely you would make some mistakes like a typo somewhere, so you would need to debug then repeat.

## Resources to debug ##

Two web resources have help me immensely:

 * [mxtoolbox](https://mxtoolbox.com/domain). Paste in your domain name it will check many things, including 100+ deny-lists. You should check here at least once per month to make sure you are not listed in any deny-list. If so, you would need to resolve the issue.
 * [mailtester](https://www.mail-tester.com/). Send an email to a automatically generated email address that the website gives you, and it will check many things, including DKIM compliance and your mail format. You need to score a perfect 10/10 here. Curiously my email through sendgrid only scored 6.4/10. Life is not fair for being an independant email sender!

## Afterword ##

It is not easy to setup a perfect email sending environment. It is also hard to keep it that way. Among other things, some of your users could have their account compromised, and forwrd a lot of SPAMs so your domain ends up in a deny-list. How to delist yourself or how to defense against such attack is out of the scope of this blog post. On the other hand, seeing some SPAMs at your end is not that bad. Let me end this blog with a quote from [Jon Postel](https://en.wikipedia.org/wiki/Jon_Postel), one of the founding fathers of the Internet:

> Be conservative in what you send, and liberal in what you accept.
