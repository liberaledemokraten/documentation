---
title: 'Spam Protection'
---

## SpamAssassin
Vesta comes with the ability to integrate [SpamAssassin](https://spamassassin.apache.org/) to classify and block spam. So, just use it. Afterwards, you may enable e.g. following black and whitelists:

* [URIBL](http://uribl.com/usage.shtml)
* [Mailspike](https://www.mailspike.net/usage.html)
* [DNSWL](https://www.dnswl.org/?page_id=15)
* [Spamhaus](https://github.com/spamhaus/spamassassin-dqs)
* [Abusix Mail Intelligence](https://docs.abusix.com/105726-setup-abusix-mail-intelligence/ami%2Fsetup%2Fspamassassin)
* [Spam Eating Monkey](https://spameatingmonkey.com/services)
* [NordSpam](https://www.nordspam.com/usage/)
* [SPFBL](https://spfbl.net/en/)
* [DKIMwl](https://www.dkimwl.org/usage)

If you utilize multiple black and whitelists, you may also want to modify the scores in a way that a single match does not result in classifying a message as spam, but multiple matches do. For instance, if your SpamAssassin spam score is set at 5.0 (default value), you may go for a configuration like this:

```
score URIBL_BLACK 3.0
score URIBL_GREY 1.5
score URIBL_RED 1.0
score URIBL_BLOCKED 0
score RCVD_IN_MSPIKE_BL 3.0
score RCVD_IN_MSPIKE_WL -2.0
score RCVD_IN_DNSWL_LOW  -1
score RCVD_IN_DNSWL_MED -5
score RCVD_IN_DNSWL_HI -20
```

Note that you should modify the Spamhaus scores in the `sh_scores.cf` and `sh_hbl_scores.cf` files instead of in a separate file if you are using the above linked Spamhaus lists.

!!! The above scores are just some sample values. We are not using the same score values per se, especially as one may have to adjust the scores depending on how effectively which rules mark spam as spam and inhowfar there are false positives. To achieve that, feedback from your users might be useful.

## Exim
### Deny Spams
You may want to modify the basic configuration which adds a spam flag to the mail to add a rule to deny messages that exceed a certain spam score. To achieve that, add the following to `acl_check_rcpt` right below the ruleset setting the spam score headers but within the `ifdef`:

```

deny
    condition      = ${if > {$spam_score_int}{500} }
    message        = Message rejected by recipient server.
    logwrite       = Message rejected. SpamAssassin detected spam with score $spam_score_int (from $sender_address to $recipients).

```

The above entry denies any messages that exceed a spam score of 50. To be honest, any such mail is most likely malicious anyways unless you had set the spam scores in a way that just a few matches would already exceed a score of 50.

### Simple Greylisting
!!! There are more advanced greylisting tools, see [Greylisting links](http://projects.puremagic.com/greylisting/links.html) for further information.

Greylisting is a way of blocking incoming spam on the assumption that bulk-spam senders won't act RFC-compliant on temporary 45x errors and not retry sending the message after more than 5 minutes. That would unnecessarily increase both memory and processor load, wasted ressources that could have been used spamming more (other) recipients. That is why it is likely that such spammers won't bother to wait a bit and retry, but skip you instead.

Unlike a proper greylisting tool, we won't look into the triplet of the sender's IP, address and the recipient address. Instead, we focus on the sender's address. In our system, this type of greylisting is not used as a replacement but as an addition to our spam filter. Within our system, senders will be greylisted in rare circumstances, so the greylist file won't be a large one. Also, we have a rather small environment without tens of thousands of accounts. That is why this primitive way of greylisting is sufficient. In a larger environment where the same sender may send bulk messages to multiple recipients, the below implementation is a terrible idea. In that case, skip the rest of this document and click on the link above showing you proper greylisting implementations.

Without further ado, that's what we add in the exim confinguration:

```

# defer if not previously greylisted
defer  !authenticated = *
    condition     = ${if and{\
                           { exists{/etc/exim4/grey.list} }\
                           { !eq {grey} {${lookup{$sender_address}lsearch{/etc/exim4/grey.list}}} }\
                     }}
    logwrite       = Info - Greylisted FROM $sender_address:grey
    message        = 4.7.1 Greylisted, please try again later
    
# accept if previously greylisted
accept   !authenticated = *
    condition     = ${if and{\
                           { exists{/etc/exim4/grey.list} }\
                           { eq {grey} {${lookup{$sender_address}lsearch{/etc/exim4/grey.list}}} }\
                     }}
    logwrite       = Sender was previously greylisted, now trusting for the rest of the day
````

Of course, you can add other types of checks before, e.g. if you have a whitelist. That check would look analogous to how the greylist is checked above.

Now, that above configuration tells yet unknown senders to try again, but there's no greylist yet. That is also why the condition will result in "false" as the file does not exist. So, let us get to how we plan to create it: through a very simple shell script and cronjobs.

This is how our primitive script looks like:

```
#!/bin/sh
grep 'Info - Greylisted FROM' /var/log/exim4/mainlog | awk '{print $NF}' > /etc/exim4/grey.list
grep 'Info - Greylisted FROM' /var/log/exim4/mainlog.1 | awk '{print $NF}' >> /etc/exim4/grey.list
```

You may have to `chown` and `chmod` in a way your exim installation can read the file. Usually, `chmod 660` should be sufficient. After having done so, feel free to add a cronjob. Note that the cronjob shouldn't run any quicker than every 5 minutes. I'd advise a timeframe between 10 and 30 minutes.

Within our greylist, all greylisted addresses from both the past and present day will be included. The `/etc/exim4/grey.list` file will ultimately look like this:

```

jane@example.org:grey
mike@example.com:grey
```

### System Filter
At times SpamAssassin fails, we still want some basic filter to run and block obvious spam. This can be realized through system-wide message filtering. See the sections [System-Wide Message Filtering](https://www.exim.org/exim-html-current/doc/html/spec_html/ch-systemwide_message_filtering.html) and [Exim Filter Files](https://www.exim.org/exim-html-current/doc/html/spec_html/filter_ch-exim_filter_files.html) within the exim documentation for further details on this.

The basic principle is that we create a system filter file. In our case it will be placed in `/etc/exim4/system_filter.conf`. The file may look like this:

```exim

# Exim filter
if
$h_subject: contains "Make money" or
$h_precedence: is "junk" or
$sender_address matches "(spammer|spambot)@"
then
  fail text "Message rejected by recipient server with status 5.7.28. Contact postmaster[at]liberale-demokraten.de for further details."
  seen finish
endif
```

Now we need to tell exim to read this file. To do so, add the following to the main configuration:

```

system_filter = /etc/exim4/system_filter.conf
```