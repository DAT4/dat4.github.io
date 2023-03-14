---
title: "Install SMTP server on Debian 11"
date: 2023-03-13T22:36:18+01:00
---

This article will show how to install a SMTP server on Debian 11. 

# Prerequisites

- Debian 11
- A domain name

# Step 1: Setup SPF in DNS correctly

You need to set up SPF and MX record to avoid getting blacklisted by other SMTP servers.

If you are using the root domain you use an `@` symbol else write then name of the subdomain. 10800 is the TTL (time to live) and then we need to set 3 records.

```
@ 10800 IN TXT "v=spf1 mx ~all"
@ 10800 IN MX 0 mydomain.tld.
@ 10800 IN A 1.2.3.4
```

- The `A` record points to our IPv4 address.
- The `MX` record states that our mailserver exist on the domain `mydomain.tld`.
- The `TXT` record states that we are using spf on all our MX records.


# Step 2: Setup the server configuration

We need to setup the server correctly.

If your domain is `mydomain.tld` then your hostname should be `mydomain`

``` 
# echo mydomain > /etc/hostname
```

Go to `/etc/hosts` and add the following line


```bash
127.0.1.1 mydomain.tld
```

Restart the computer

# Step 3: Install postfix

```
# apt install postfix
```

Chose all the default settings

The postfix service should automatically be enabled and started - if this is not the case you can do it.

```
# systemctl enable postfix --now
```

# Step 4: Test if it works

Install netcat

```
# apt install netcat
```

```
nc 127.0.0.1 25 << EOF                     ~
HELO localhost
MAIL FROM: yourname@mydomain.tld
RCPT TO: address@gmail.com
DATA
Subject: This is a test

Hello this is a test to see if the mailserver works.

.
QUIT
EOF
```

If it doesn't work then it could be becaue your domain is blocked or because a firewall is blocking outgoing smtp traffic.

You can troubleshoot the mailserver with 

```
# journalctl -f -u postfix
```

and read the logs for mails here:

```
# tail -f /var/log/mail.log
```

in both cases the `-f` flag is to `follow` the file and show any inputs arriving while reading.

# Step 5: Configure postfix

Most configuration in postfix can be done in `/etc/postfix/main.cf`

## Restrict mail from to come from your domain

This is good to avoid getting blocked for abuse for using other email addresses.

Create a file `/etc/postfix/allowed_senders`

```
/@mydomain.tld$/ OK
// REJECT
```

and add this to `main.cf`

```
smtpd_sender_restrictions = check_sender_access regexp:/etc/postfix/allowed_senders
```

Now the server will throw an error whenever someone uses another email address than the allowed one in the `MAIL FROM` field.


# Some hacks

By default you can only send emails from localhost - I have not figures out how to properly set this up. But a quick hack is to remoteforward port 25 with SSH. Then postfix will always think that you are sending from localhost.

```
# apt install open-ssh
# systemctl enable ssh --now
# ssh -R 2525:localhost:25 localhost
```

You can make a system service so that this always happens

create this file `/etc/systemd/system/smtphack.service`
```
[Unit]
Description=SSH remote forward SMTP server hack
After=network.target

[Service]
Type=simple
ExecStart=/usr/bin/ssh -NT -o ExitOnForwardFailure=yes -o ServerAliveInterval=60 -R 2525:localhost:25 localhost
RestartSec=5
Restart=always

[Install]
WantedBy=multi-user.target
```

```
# systemctl daemon-reload
# systemctl enable smtphack --now
```





