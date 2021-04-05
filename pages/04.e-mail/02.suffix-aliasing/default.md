---
title: 'Suffix Aliasing'
---

By enable suffix-aliasing, one can set e.g. `mail+whatever@example.com` as an alias of `mail@example.com`. That way, all mails to `mail+somethingelse@example.com` will be received in the same inbox as `mail@example.com`. As you noticed, it does not matter what the plus sign is followed by. To enable this, go through the following steps:

1. Log into Vesta CP as admin
2. Go to "server" and the configuration of exim
3. Go to the section "ROUTERS CONFIGURATION" and find `localuser` preferences

There, add the following lines:

```exim
localuser:
  driver = accept
  transport = local_delivery
  condition = ${lookup{$local_part}lsearch{/etc/exim4/domains/$domain/passwd}{true}{false}}
  local_part_suffix = +* : #* : =* : &*
  local_part_suffix_optional
```

This will result in enabling suffixes prepended by either of the following characters: +. #, =, and &.