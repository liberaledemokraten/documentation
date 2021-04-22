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
You may want to modify the basic configuration which adds a spam flag to the mail to add a rule to deny messages that exceed a certain spam score. To achieve that, add the following to `acl_check_rcpt` right below the ruleset setting the spam score headers but within the `ifdef`:

```

deny
    condition      = ${if > {$spam_score_int}{500} }
    message        = Message rejected by recipient server.
    logwrite       = Message rejected. SpamAssassin detected spam with score $spam_score_int (from $sender_address to $recipients).

```

The above entry denies any messages that exceed a spam score of 50. To be honest, any such mail is most likely malicious anyways unless you had set the spam scores in a way that just a few matches would already exceed a score of 50.