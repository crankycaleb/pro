---
layout: post
title: Backing Up Fedora 28 with Duplicati 2
category: Backup
tags:
- duplicati
- backup
- fedora
- nextcloud
subtitle: Simple and Powerful Encrypted Backups
---

(REVISED WITH NGINX SSL CONFIG! Thanks for the idea, [SpryServers](https://twitter.com/SpryServers)!

Personally, one of the scariest parts of self-hosting is having all my data under one roof. Power issues, theft, fire or other natural disasters are just a few of the potential risks to my data. Luckily, the open source project [Duplicati](https://www.duplicati.com/) has proven to be a reliable option. Open Source, AES 256 encryption, and a wide variety of backup destinations supporting anywhere from Webdav, S3, to OneDrive, Mega, Google Drive, Dropbox, and others. This solution has given me offsite backups that are encrypted, while also leveraging the proprietary storage provider space I've accumulated (Samsung 100GB OneDrive storage for two years promos, etc) with no cost and minimal risk of data snooping.

This post is by no means intended to be a detailed Duplicati informative guide, as their website already provides plenty of documentation. I'm documenting my own experience that myself or others can use as a "Getting Started" gritty reference to build the basics of a functioning backup solution in Fedora 28 with emailed logs. Let's begin.


### 1.) Duplicati 2 Setup:

Install the prerequisites:

```
sudo dnf install mono-core libappindicator libappindicator-sharp desktop-file-utils
```

Grab the latest rpm (check their site from a web browser first):

```
wget https://updates.duplicati.com/beta/duplicati-2.0.3.3-2.0.3.3_beta_20180402.noarch.rpm
```

Install the rpm:

```
sudo rpm -ivh duplicati*
```

Poke holes in firewalld to allow access to the web management frontend:

```
sudo firewall-cmd --permanent --add-port=8200/tcp
```

and then:

```
sudo firewall-cmd --reload
```

Start Duplicati and tell it to listen on interfaces other than localhost:

```
sudo duplicati-server --webservice-interface=any
```

Leave running in terminal, go to the web interface from another pc (http://serverip:8200), set a password, then go back to terminal and ctrl-c.

To work with many third party services, you're going to need to run a cert-sync after setup:

```
sudo cert-sync /etc/pki/tls/certs/ca-bundle.crt
```

By default, Duplicati expects a graphical desktop and is started by a tray object. I'm running a headless nextcloud instance in this example, so I need to be able to run this as a service instead. Let's grab a service config:

```
wget https://raw.githubusercontent.com/duplicati/duplicati/master/Installer/debian/debian/duplicati.service
```

Move it:

```
sudo mv duplicati.service /etc/systemd/system
```

Then enable at startup and start the service:

```
sudo systemctl enable duplicati
```

and then:

```
sudo systemctl start duplicati
```

Now to move to configuring for our needs via the web interface.
****
### 2.) Configuring a Backup Job:

Duplicati runs a web service on port 8200 with no native ssl support. We'll add SSL support at the end of this document, we just want to make sure we get everything else working first.

The web frontend is fairly self explanatory (and mobile friendly!) as shown in some of their own screenshots:

![Desktop](/img/Duplicati-UI.jpg)

![Mobile](/img/Duplicati-mobile.jpg)

![Picker](/img/Duplicati-SourcePicker.jpg)

There are too many backup destination provider options to cover here, so I recommend you check their site for documentation on the one(s) you plan to use. A good place to start with configuring your first backup is [here.](https://www.duplicati.com/articles/Getting-Started/) The backup job setup process is simple so I'm going to provide the email notification flags you'll want to add to cover all of your backup jobs.

Go to the main Settings section. You'll want to add your email service settings (of which you'll need to know) in the main settings section so you don't have to specify them manually as additional flags to every single job you create. These will be "advanced" additions you'll select from the dropbox, add, and configure one at a time. 

**send-mail-to:**

--send-mail-to (String)
Email recipient(s).
This setting is required if mail should be sent, all other settings have default values. You can supply multiple email addresses separated with commas, and you can use the normal address format as specified by RFC2822 section 3.4.
Example with 3 recipients: Peter Sample <peter@example.com>, John Sample <john@example.com>, admin@example.com

**send-mail-from:**

--send-mail-from (String)
Email sender.
Address of the email sender. If no host is supplied, the hostname of the first recipient is used. Examples of allowed formats:

    sender
    sender@example.com
    Mail Sender <sender>
    Mail Sender <sender@example.com>

**send-mail-subject:**

--send-mail-subject (String)
The email subject.
This setting supplies the email subject. Values are replaced as described in the description for --send-mail-body.
Default value: Duplicati %OPERATIONNAME% report for %backup-name%

Of note I like to add the %PARSEDRESULT% variable to the beginning of the subject line so I can quickly see success or failure.

**send-mail-url:**

--send-mail-url (String)
SMTP Url.
A url for the SMTP server, e.g. smtp://example.com:25. Multiple servers can be supplied in a prioritized list, separated with semicolon. If a server fails, the next server in the list is tried, until the message has been sent.
If no server is supplied, a DNS lookup is performed to find the first recipient's MX record, and all SMTP servers are tried in their priority order until the message is sent.
To enable SMTP over SSL, use the format smtps://example.com. To enable SMTP STARTTLS, use the format smtp://example.com:25/?starttls=when-available or smtp://example.com:25/?starttls=always. If no port is specified, port 25 is used for non-ssl, and 465 for SSL connections. To force not to use STARTTLS use smtp://example.com:25/?starttls=never.

**send-mail-username:**

--send-mail-username (String)
SMTP Username.
The username used to authenticate with the SMTP server if required.

**send-mail-password:**

--send-mail-password (String) SMTP Password.
The password used to authenticate with the SMTP server if required.

**send-mail-level:**

--send-mail-level (String)
The messages to send.
You can specify one of Success, Warning, Error, Fatal. You can supply multiple options with a comma separator, e.g. Success,Warning. The special value All is a shorthand for Success,Warning,Error,Fatal and will cause all backup operations to send an email.
Values: Unknown, Success, Warning, Error, Fatal, All
Default value: all

Hopefully this can get you started with the basics and just go from there. Of note to the nextcloud world, I was sure to already have the OwnBackup app installed and doing scheduled database backup dumps. It stores those in your /data directory, so will be included in your backups already whenever you select that as one of the backup source locations.
****
### ADDENDUM: Adding SSL
****
Now that you've tested and have a working backup setup, let's lock the web interface down with SSL by setting up an nginx reverse proxy. We'll start by installing nginx:

```
sudo dnf install nginx
```

Then we'll need to navigate to /etc/nginx/ to rm the default nginx.conf and replace it with one that won't conflict with our nextcloud installation:

```
wget https://raw.githubusercontent.com/crankycaleb/scripts/master/nginx.conf
```

We need to edit this file in your favorite editor, and replace `yourfqdn` with your actual fully qualified domain name in the following 4 lines:

(Line 41)
```
server_name  yourfqdn;
```

(Line 44)
```
ssl_certificate "/etc/letsencrypt/live/yourfqdn/cert.pem";
```

(Line 45)
```
ssl_certificate_key "/etc/letsencrypt/live/yourfqdn/privkey.pem";
```

(and Line 64)
```
proxy_redirect	http://localhost:8200 https://yourfqdn:8443;
```

Then save and exit your editor. Next we'll want to start nginx:

```
sudo systemctl start nginx
```

and enable it at boot

```
sudo systemctl enable nginx
```

Now we certainly can't forget to poke a hole in our firewall for it:

```
sudo firewall-cmd --permanent --add-port=8443/tcp
```

Remove the plain http hole:

```
sudo firewall-cmd --permanent --remove-port=8200/tcp
```

And reload our firewall:

```
sudo firewall-cmd --reload
```
****
And now you should have a fully working SSL encrypted web frontend for Duplicati 2 when you browse to https://yourfqdn:8443. This howto assumes you followed my other howto's and have configured letsencrypt. If you have not done so and prefer to use your own self signed certificates, you'll need to adjust those lines in nginx.conf accordingly.
