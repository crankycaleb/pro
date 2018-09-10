#---
#layout: post
#title: Backing Up Fedora 28 with Duplicati
#category: Backup
#tags:
#- duplicati
#- backup
#- fedora
#subtitle: Simple and Powerful Encrypted Backups
#---

Personally, one of the scariest parts of self-hosting is having all my data under one roof. Power issues, theft, fire or other natural disasters are just a few of the potential risks to my data.
Blah Blah backups are good AES 256 encrypted

**Duplicati Setup**

BlahBlah Fedora start here wget the rpm etc:

`sudo dnf install mono-core libappindicator libappindicator-sharp desktop-file-utils`
`wget https://updates.duplicati.com/beta/duplicati-2.0.3.3-2.0.3.3_beta_20180402.noarch.rpm`
`sudo rpm -ivh duplicati*`
`sudo duplicati-server --webservice-interface=any`
`sudo firewall-cmd --permanent --add-port=8200/tcp`
`sudo firewall-cmd --reload`
leave running in term go to web interface, set password, then go back to term ctrl-c
`sudo cert-sync /etc/pki/tls/certs/ca-bundle.crt`
`wget https://raw.githubusercontent.com/duplicati/duplicati/master/Installer/debian/debian/duplicati.service`
`sudo mv duplicati.service /etc/systemd/system`
`sudo systemctl enable duplicati`
`sudo systemctl start duplicati`



Then run:

`sudo blah`

Blah blah blah

**The Minor Gotchas**
