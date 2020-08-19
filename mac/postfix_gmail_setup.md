---
permalink: /postfix_gmail_setup/
---

# Configure Postfix for Gmail SMTP in Mac OSX

Edit file `/etc/postfix/main.cf` and add this to the bottom:

```
# Configure Postfix for Gmail SMTP in Mac OSX Yosemite
# Added per https://gist.github.com/joech4n/72108461bfac1bf2e99f
# Set the relayhost to the Gmail Server.  Replace with your SMTP server as needed
relayhost = [smtp.gmail.com]:587
# Postfix 2.2 uses the generic(5) address mapping to replace local fantasy email
# addresses by valid Internet addresses. This mapping happens ONLY when mail
# leaves the machine; not when you send mail between users on the same machine.
smtp_generic_maps = hash:/etc/postfix/generic

# These settings (along with the relayhost setting above) will make
# postfix relay all outbound non-local email via Gmail using an
# authenticated TLS/SASL session.
smtp_tls_loglevel=1
smtp_tls_security_level=encrypt
smtp_sasl_auth_enable=yes
smtp_sasl_password_maps=hash:/etc/postfix/sasl/sasl_passwd
smtp_sasl_security_options = noanonymous

# To fix these errors per http://askubuntu.com/q/73865:
# Dec 15 17:14:12 localhost.local postfix/smtp[3691]: Untrusted TLS connection established to smtp.gmail.com[74.125.28.108]:587: TLSv1 with cipher RC4-SHA (128/128 bits)
smtp_tls_CApath = /usr/local/etc/openssl/certs
smtp_tls_CAfile = /usr/local/etc/openssl/cert.pem

# To fix these errors per http://stackoverflow.com/q/26447316:
# Dec 15 17:46:51 heimerdinger.local postfix/smtp[4758]: C9682156786: to=<username@gmail.com>, relay=smtp.gmail.com[74.125.28.108]:587, delay=1.3, delays=0.77/0.11/0.42/0, dsn=4.7.0, status=deferred (SASL authentication failed; cannot authenticate to server smtp.gmail.com[74.125.28.108]: generic failure)
smtp_sasl_mechanism_filter = plain
```

## Create a sasl_passwd if one doesn't exist

```bash
sudo mkdir /etc/postfix/sasl
sudo vim /etc/postfix/sasl/sasl_passwd
```

and enter in the following:

```
[smtp.gmail.com]:587 username@gmail.com:password
```
**NOTE**: If you use two factor auth on your account you will need to generate a Google App Password and put that in place of your normal password.

## Set up address mapping
Use the generic(5) address mapping to replace local fantasy email (user@host.local) addresses by valid Internet addresses (username@gmail.com). This mapping happens ONLY when mail leaves the machine; not when you send mail between users on the same machine. Set this up by editing `/etc/postfix/generic`.

```bash
sudo vi /etc/postfix/generic
```

and add the following (only think you need to replace is `GMAIL_USERNAME`:

```
user@host.domain GMAIL_USERNAME@gmail.com
@host.domain     GMAIL_USERNAME@gmail.com
```

## Protect credentials, create Postfix files, and restart Postfix

```bash
sudo chmod -R 600 /etc/postfix/sasl
sudo postmap /etc/postfix/sasl/sasl_passwd
sudo postmap /etc/postfix/generic
sudo postfix stop
sudo postfix start
```

## Testing

```bash
echo 'test' | mail -s "contents" your@yourdomain.com
```

## Errors?

If you receive the following error:

```bash
send-mail: fatal: chdir /Library/Server/Mail/Data/spool: No such file or directory
```

you can do the following:

```bash
sudo mkdir -p /Library/Server/Mail/Data/spool
sudo /usr/sbin/postfix set-permissions
sudo /usr/sbin/postfix start
```

as per [this question](http://apple.stackexchange.com/questions/54051/sendmail-error-on-os-x-mountain-lion).

**NOTE**: If things aint sending / receiving, and you're getting notices, check that the mail servers you're using are actually working!

**NOTE**: You can check the mail queue using `mailq` command.

**NOTE**: You can check the mail log by viewing `/var/mail/USERNAME`

