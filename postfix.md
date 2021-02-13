# Postfix as a Send Only SMTP Server

https://www.digitalocean.com/community/tutorials/how-to-install-and-configure-postfix-as-a-send-only-smtp-server-on-ubuntu-20-04

- `apt update`

mailutils for testing purposes.
- `apt install mailutils postfix`

Postfix configuration window will appear. 
- Select **Internet Site**.
- System mail name should be the domain name (`example.com` and not `smtp.example.com`)

- Edit `/etc/postfix/main.cf`

`inet_interfaces = loopback-only`

`mydestination = localhost.$mydomain, localhost, $myhostname`

`myhostname = $mydomain` ?


Can ignore this part at first, to see if it even works without TLS.

```
# Note: Settings starting with "smtpd_" are for emails that are send to the server. Settings starting with "smtp_" are for emails that are sent to other servers.
smtp_tls_CAfile=/etc/path/to/certificate/cert.pem
smtp_tls_cert_file=/etc/path/to/certificate/fullchain.pem
smtp_tls_key_file=/etc/path/to/certificate/privkey.pem
smtp_use_tls=yes
```

- Check if the following files contain your domain name
`/etc/hostname`

and

`/etc/mailname`


- Restart postfix

`systemctl restart postfix`

- Test sending emails

`echo "This is the body of the email" | mail -a "From: user@yourdomain.com" -s "This is the subject line" your_email_address`

- Check header of received email for something about TLS being used

# Synapse SMTP Settings

```
email:
  smtp_host: localhost
  require_transport_security: true
  notif_from: matrix@yourdomain.com
  notif_for_new_userse: false
  invite_client_location: https://app.element.io
