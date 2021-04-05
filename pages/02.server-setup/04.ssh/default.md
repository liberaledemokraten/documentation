---
title: SSH
taxonomy:
    category:
        - ssh
    tag:
        - configuration
---

We will be using SSH for both file transfers as well as managing the server remotely. In fact, the installation of the server management plattform has required SSH access too. This is a highly crucial matter as alot can go wrong if unauthorized people could gain access via SSH. Therefore, there will be a few security rules.

## Passwordless Login
The `root` user must have to log in without any password but through an RSA, ECDSA or ed25519 key. Other users too should not have access using a passphrase, if possible. But as it's not expectable for every branch to manage logging in using keys and storing them safely, it is also okay to allow login using passphrases.

To achieve passwordless login in general, first open the SSH configuration file:

```
vim /etc/ssh/sshd_config
```

Then find the line containing `PasswordAuthentication no` and uncomment it. Now, no user may log in using a password or passphrase. If you however want to disable password authentication for a single user (e.g. root), go as follows:

```
Match User root
    PasswordAuthentication no
```

An even better and more recommended configuration is this:

```
Match Group root,sudo
    PasswordAuthentication no
```


## Empty Passwords
There may be some users created without passwords having been set. In general, that's a bad thing and should never be done. In fact, new users shouldn't be created in the console just out of the blue and any created additional user outside of Vesta should be documented seperately. But the worst thing is if we allow them to login using no password at all. That's a huge security issue and should be avoided at all cost.

To do so, open the configuration file as explained above and make sure the following directive is enabled:

```
PermitEmptyPasswords no
```

If you want to check out what users are on your system, you can have a look into the `/etc/passwd` file. Alternatively, you can run

```
awk -F: '{ print $1 " " $2}' /etc/passwd
```

This will display all users on your system and their passwords, or an `x` in case the passwords are stored in `/etc/shadow` in an encrypted form.


## Limit Login Attempts
If you open the configuration file, you'll see `MaxAuthTries`. Feel free to uncomment it. The more recommended approach however is to use fail2ban to deal with that. If you log into Vesta as an administrative user, you'll see "Server" at the very top. There, search for fail2ban and modify its configuration. The recommended setting is:

```
[ssh-iptables]
enabled  = true
filter   = sshd
action   = vesta[name=SSH]
logpath  = /var/log/auth.log
maxretry = 3
```

This will block the IP address of anyone once three failed attempts were made. The IP adresses can however be unblocked from the management console, if necessary. So using fail2ban gives more administrative power and flexibility here.

## Login Notification
We may want to know of successful SSH logins. It will help keeping track of when who has logged in, but it also helps us being notified in case somebody possibly unauthorized gains access to the server. To achieve that, do as follows:

1. `apt-get install mailx`
2. Add the following line to `~/.bashrc`

```
echo -e "Subject: Alert - Shell Access\nFrom: noreply@example.com\nTo: someone@example.com\n\nAlert - Shell Access to `hostname` on `date` by `whoami` `who` via `echo $SSH_CLIENT | awk '{ print $1}'`" | sendmail -t
```