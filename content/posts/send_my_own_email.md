---
title: Send My Own Emails
slug: send_my_own_emails
date: 2020-08-06
tags:
- email
description: Sending your own email by yourself is suprisingly hard.
---

I've been running my email server for a year now. I have my domain, my email addresses, my email server, so if you send me an email, it will go through the least possible number of hops into my brain. However, up until now, I send my emails through a 3rd party vendor, [SendGrid](https://sendgrid.com/), for the simple reason which is to reach my recipients reliably. One very prominent email provider that starts with the letter G has caused me the most troubles. With Sendgrid doing the deliveries, all the worries are theirs.

## Why switch ##

My email sending needs are modest. The free plan from Sendgrid is good enough for me, and I could afford the lowest paid plan if I have to. However, it still feels wrong to use Sendgrid, other than me being cheap:

 * My email server is perfectly capable of sending my emails to the final destinations. Why should my emails take a detour through a vendor, who can read or even alter my emails?
 * I am not among the targeted customers for Sendgrid (or Mailgun, Mailchimp, etc.). They boast about email campaigns and newsletters for companies, while I am just a guy sending emails for myself, my friends, and my family.
 * Sendgrid has certain "features" that make me uncomfortable. For example, it can turn the links in my emails into [spying links](https://www.campaignmonitor.com/resources/knowledge-base/what-is-click-through-rate-how-can-ctr-be-calculated/), or insert the infamous [spying pixel](https://en.wikipedia.org/wiki/Web_beacon). Yes, I opted out all of the above; however, I feel wrong by being their customer, even on a free plan. 

## The pain of sending emails directly ##

Sending emails is supposed to be easy: The email server looks up the MX record, figured out which server to contact then just shot the email over. However, my previous tries of sending emails myself have resulted in a lot of bounced emails, or emails landing in the "junk" folder. Why? And why I can't go through by myself while Sendgrid can? Yesterday I sat down and determined to solve the mysteries. Here are what I have figured out.

### IPv6 ###

The first thing is simple but not obvious: Disable IPv6 in your email server software [exim](https://exim.org/). I know IPv6 is supposed to solve the Internet problem, and everyone should support it. However, if an email server does not have an IPv4 address, it cannot receive many emails. So IPv6 does not give you any benefit while giving you twice the debugging headache. In my case, merely having IPv6 turned on will cause trouble talking to some hosts, thanks to my registrar [domain.com](https://www.domain.com/)'s lack of proper [AAAA DNS records](https://support.dnsimple.com/articles/aaaa-record/).

### Basic DNS and rDNS ###

I had them all along, but here I want to reiterate the basic DNS setup because of their importance. You should have the following at the minimum:

 * An A record for your domain.
 * An A record for the email server of your domain.
 * A MX record for your domain that point to your email server's domain name
 * Reverse DNS records from both IPv4 and IPv6 addresses of your email server that point to your email server's domain name
 
### Web server and SSL ###

You should have a web server on your email server, even if you don't need one. Just make one serving a blank page, if nothing else. And that web server should have a proper SSL certificate (not self-signed), so https works. [Certbot](https://certbot.eff.org/) will help you achieve this.

Your email server should also have TLS (not self-signed) on SMTP. The certificate you get for the webserver comes in handy now. You also want to add a deploy hook in your Cerbot configuration to reload your email server after the deployment of a new certificate. 

### SPF ###

You must have [Sender Policy Framework](https://en.wikipedia.org/wiki/Sender_Policy_Framework). It is a DNS entry that publishes an allow-list that tells the world which IP can send your emails. For simplicity's sake, just set it to: `v=spf1 a mx ~all`. It lists 2 IPs, one from the A record of your domain, the other from the A record that is referenced by the MX record of your domain. All the rest are disabled. All emails that originate from your domain must come from either one of them.

### DKIM ###

[DKIM](https://en.wikipedia.org/wiki/DomainKeys_Identified_Mail) is another way to prove the authenticity of your email. It feels redundant now that you have SPF, but you must have both of them. DKIM is a DNS entry that contains a public key. The receiving email servers can verify the signature that your email server embedded in the header of outgoing emails. The setup is a little complicated, so I document here in detail, using Exim4 on Debian as an example. If you want to follow, please use your domain name instead of `3qin.us`.

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

Remember, the conf file must end with a newline. This problem has burned me.

#### Install the public key in DNS ####

In your DNS you should have a TXT entry named `20200803._domainkey`, which is the same string as your DKIM_SELECTOR appended with `._domainkey`. The value should be `k=rsa; VERYLONGUGLYPUBKEY` where the VERYLONGUGLYPUBKEY is your public key in one non-breaking string. Make sure you copied and paste the full key without any space or carriage return in the middle!

### DMARC ###

[DMARC](https://en.wikipedia.org/wiki/DMARC) is designed to tell email domain owners what to do if the SPF or the DKIM check failed. Logically you'd think there should be a sane default, but no, you have to have the rules spelled out. For simplicity's sake, add one TXT entry named `_dmarc` with value as:

```
v=DMARC1; p=quarantine; sp=reject; rua=mailto:postmaster@3qin.us
```

It tells domain owners to quarantine (put in the junk folder) everything that failed either SPF or DKIM from your domain (you could have misconfigured something). And to bounce everything that failed either SPF or DKIM from a subdomain of your domain (I don't have such things, so they must be bogus). It also tells the domains to send a periodic summary report to the postmaster email of your domain. (Yeah, I am going to read that.)

That's all, wait for your DNS records to propagate, then test. Most likely, you made some mistakes like a typo somewhere, so you would need to debug then repeat.

## Resources to debug ##

Two web resources have helped me immensely:

 * [mxtoolbox](https://mxtoolbox.com/domain). Paste in your domain name it will check many things, including 100+ deny-lists. You should check here at least once per month to ensure your domain or IP is not in any deny-list. If so, you would need to resolve the issue.
 * [mailtester](https://www.mail-tester.com/). Send an email to an automatically generated email address that the website gives you, and it will check many things, including DKIM compliance and your mail format. You need to score a perfect 10/10 here. Curiously my email through Sendgrid only scored 6.4/10. Life is not fair for being an independent email sender!

## Afterword ##

It is not easy to set up a perfect email sending environment. It is also not easy to keep it that way. Some of your users could have their account compromised and forward a lot of SPAMs so your domain could end up in a deny-list. How to delist yourself or defend against such attacks is out of the scope of this blog post. 

On the other hand, seeing some SPAMs at your end is not that bad. Let me end this blog with a quote from [Jon Postel](https://en.wikipedia.org/wiki/Jon_Postel), one of the Internet's founding fathers:

> Be conservative in what you send and liberal in what you accept.

If only the big email providers are doing the same.
