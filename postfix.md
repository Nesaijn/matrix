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

# OpenDKIM

https://wiki.debian.org/opendkim

- Install the necessary packages

`apt install opendkim opendkim-tools`

- Generate the key as the opendkim user

The selector (-s) is just a part of the subdomain where the TXT entry is later being made and the files are named like that. Could also be like `mail`.

`sudo -u opendkim opendkim-genkey -D /etc/dkimkeys -d yourdomain.com -s 2021`

- Edit the `/etc/opendkim.conf`

Configure the following parameters as follows:

```
Domain   yourdomain.com
Selector 2021
KeyFile  /etc/dkimkeys/2021.private

Socket   inet:8891@localhost
```

- Restart OpenDKIM

`systemctl restart opendkim`

- Edit Postfix to use OpenDKIM. In `/etc/postfix/main.cf`

`smtpd_milters` matching the Socket which is configured in `/etc/opendkim.conf`

```
smtpd_milters = inet:localhost:8891
non_smtpd_milters = $smtpd_milters
```

- Reload the Postfix configuration

`systemctl reload postfix`

- Publish the public key as TXT record in DNS

There is a .txt file in `/etc/dkimkeys/` named like the choosen selector (`2021.txt`).
In that file is the public key which has to be served. It also says where.
The file may look like following

```
2021._domainkey	IN	TXT	( "v=DKIM1; h=sha256; k=rsa; "
	  "p=SOMERANDOMCHARACTERSSOMERANDOMCHARACTERSSOMERANDOMCHARACTERS...
SOMERANDOMCHARACTERSSOMERANDOMCHARACTERS"
    "SOMERANDOMCHARACTERS/SOMERANDOMCHARACTERSSOMERANDOMCHARACTERSSOMERANDOMCHARACTERSSOMERANDOMCHARACTERS...") ; ---- DKIM key 2021 for yourdomain.com
```

Where to serve the key is shown at the beginning. In this case at: 
`2021._domainkey.yourdomain.com`

When creating the TXT record it probably is enough to just input `2021._domainkey` as the name/subdomain.
The data/value entry at that that name/subdomain has to be the two strings following the key specifications, concatenated:

`p=SOMERANDOMCHARACTERSSOMERANDOMCHARACTERSSOMERANDOMCHARACTERS...SOMERANDOMCHARACTERS/SOMERANDOMCHARACTERSSOMERANDOMCHARACTERSSOMERANDOMCHARACTERSSOMERANDOMCHARACTERS...`

Ignoring the key specifications and the quotes, the brackets etc.

# DMARC and SPF

Additionally you want to have two more TXT records for security reasons. (?)

- For DMARC (replace `yourdomain.com` with yours):

At `_dmarc.yourdomain.com` (`_dmarc`)

this TXT data (replace `yourdomain.com` with yours):

`v=DMARC1; p=reject; rua=mailto:dmarc@yourdomain.com; fo=1`

- For SPF (replace `yourdomain.com` and `yourmaildomain.com` with yours)

At `@.yourdomain.com` (`@`)

this

`v=spf1 mx a:yourmaildomain.com -all`

- You can test your settings at this website

https://appmaildev.com/en/dkim

Click on the "Next" button to get a temporary email adress where you send an email through your SMTP server.
When everything is correct it should show "pass" at "SPF", "DKIM" and "DMARC".


# Synapse SMTP Settings

- To use sending emails configure the following in the synapse config file `/etc/matrix-synapse/homeserver.yaml`

```
email:
  smtp_host: localhost
  require_transport_security: true
  notif_from: matrix@yourdomain.com
  notif_for_new_userse: false
  invite_client_location: https://app.element.io
```
