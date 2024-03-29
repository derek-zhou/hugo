---
title: SPAM blacklisting is out of control
slug: spam_blacklists_are_out_of_control
date: 2022-02-04
tags:
- email
description: If you got shuned by the society not because of what you did but what your neighbor did, you will cry bloody injustice. This is exactly what is happening in SPAM blacklisting.
---

Nobody likes SPAM. In the name of fighting SPAM, people have created numerous blacklists to list the known spammers’ IPs; and those blacklists are the center pieces of today’s SPAM control technology. Since [I send my own emails](/posts/send_my_own_emails/), I regularly check those blacklists to make sure my mail server’s IP is not on them, so my correspondents can receive my emails. Yesterday, I suddenly found out that my IP is blacklisted! Specifically, I am on UCEPROTECTL2 and UCEPROTECTL3. I don’t SPAM, so how can it be? Did I get pwn’ed?

It turns out that my server is fine. However, they think my IP is in a bad neighborhood: Someone else using the IPs from the same hosting company as I do is spamming. To quote them directly:

> Who is responsible for this listing?
>
> YOU ARE NOT!. Your IP XX.XX.XX.XX was NOT directly involved in abuse, but has a bad neighborhood. Other customers within this range did not care about their security and got hacked, started spamming, or were even attacking others, while your provider has possibly not even noticed that there is a serious problem. We are sorry for you, but you have chosen a provider not acting fast enough on abusers.

## So what should I do?

I see three options:

I can complain to my hosting company and hope they evict the bad user from the network. But then why should my hosting company do so? If my understanding of the law is correct, spamming is legal, albeit immoral. If I were the hosting company, and I had a paying customer that is conducting legal business on my network, on what grounds can I evict them? Also one needs to consider the severity of the offence: there is a difference between constantly spamming and occasional spamming. My hosting company has hundreds of thousands of IPs; If only one percent of the IPs are spamming occasionally, the combined volume of SPAM originated from the network would be huge and push the whole network into the blacklists. Does it make business sense, let alone is it technically feasible, to enforce a zero tolerance on spamming for a hosting company?

Or I can leave the current hosting company. My hosting company is competitively priced, is fast, and has served me well for many years. I don’t have a long term contract with them so I can switch any time. However, switching has a cost in my labor, and how would I know that the new hosting company can stay off the blacklists for long?

Finally, I can pay up. UCEPROTECT conveniently embedded a link to [http://www.whitelisted.org/](http://www.whitelisted.org/), which offers to whitelist my IP. Had I been on the whitelist, the blacklists would not concern me. To quote them:

> Since your IP wasn't directly involved in abuse, you can exclude your IP from neigborhood blocklists as UCEPROTECT Levels 2 and 3 and others that are importing our whitelist, by registering your IP with us.

There is only one problem: 

> Registration is available for 1 Month (25 CHF), 6 Month (50 CHF), 12 Month (70 CHF), 24 Month (90 CHF) 

To me this is highway robbery. 25 CHF per month is five times the amount I paid to my hosting company for the VPS! I got the cheap VPS because my hosting company is in a fierce competition with other hosting companies; how many whitelisting companies are out there? Did I sense monopoly?

## Are the blacklists that useful?

I run my own email server, and don’t use any blacklist. Yes, I got some amount of SPAMs. However, the vast majority of trashy emails I receive each day are advertisements (I signed up for random stuff quite liberally). I know I can use a blacklist and cut down the volume of unwanted emails to my inbox, but the difference won’t be much, because the Ads are sent from legitimate senders, which are not on any blacklist. Even if they were, they surely are rich enough to pay the 25 CHF per month whitelisting fee.

What is SPAM, but some form of advertisement from poor people, with questionable intentions and potentially harmful content? Now email advertising is a multi billion dollar industry, while SPAM has been demonized over the years. I can’t help to wonder if it has anything to do with the rise of big email vendors, who with no exception are Ads platforms themselves. Your “free” personal email accounts are nothing but an Ads delivery pipeline customized to exploit you. Your paid and hosted corporate email accounts are run by the same big email vendors. How much better can they be? The big email vendors love to enforce onerous rules for SPAM blocking; is it for their users, or for themselves?

The problem of SPAM control boils down to the issue of signal to noise ratio that one can tolerate. More filtering leads to higher signal to noise ratio, with collateral damages such as filtering out some real signals. To me, an intelligent user armed with a [good email client](https://github.com/derek-zhou/liv), one trash email would only cost me half a second to glimp through. On the other hand, if a legitimate email from a future friend or a potential business associate were accidentally blocked, the lost opportunity cost is several magnitudes higher. From a cost benefit analysis point of view, I cannot justify blacklisting IPs in my email setup; and I don’t think I am alone. 

If by any slim chances, you happen to be in charge of an email setup, please, don’t sink to the low of the big email vendors. IP blacklists have their place, however, please don’t use the overarching neighboring blacklists such as UCEPROTECTL2 and UCEPROTECTL3.



