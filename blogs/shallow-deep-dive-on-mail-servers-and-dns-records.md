{% head %}
{% info %}

{% title-shallow-deep-dive-on-mail-servers-and-dns-records %}

{% banner-shallow-deep-dive-on-mail-servers-and-dns-records %}

{% content-start %}

## Introduction
We use email daily, and it is the invisible but most important marketing and communication tool. We developers use email for several things, but we (at least me) don't understand how they actually work under the hood. We make some DNS changes for Resend to work but don't really check what they mean. 

My goal for this blog is to understand how mail servers and protocols work and try to explain them to you as well. So let's get started!

## How Mail Servers Work
### Key Actors
#### Mail User Agent (MUA)
Our email client like a web browser, Thunderbird, or something like Outlook. The application we use to read and compose emails.
#### Mail Transfer Agent (MTA)
This is software that transfers emails from sender to receiver using SMTP.
#### Mail Delivery Agent (MDA)
Postal mailbox equivalent for emails.
#### Mail Retrieval Agent (MRA)
Server component that allows user agents to retrieve messages using protocols like IMAP or POP3

### Sending an Email
When the send button is clicked, your client (MUA) connects to the mail server (MTA) via SMTP. The MTA then looks at the recipient's email address to locate where to send the email next. It uses the domain part (after @) to query DNS for that domain's MX record, which contains the recipient's mail server's address. After the address is located, the MTA starts to do *store and forward* - they queue the messages and retry if the next hop is unreliable, which ensures reliable connection.
### Key Protocols
- **Simple Mail Transfer Protocol (SMTP):** Used to send emails from client to server and between MTAs.
- **Extended Simple Mail Transfer Protocol (ESMTP):** An extension of SMTP that supports additional features like authentication and attachments.
- **Post Office Protocol v3 (POP3):** Downloads emails from the server to the client and usually deletes them from the server.
- **Internet Message Access Protocol (IMAP):** Syncs emails between the server and client, keeping them both on client and server - a more modern approach.
## DNS Setup for Email Service
This is the part I'm most interested in. I think we've all done this setup more than once in our lives, but most of us don't go one step further to actually understand these settings.

### MX Records - Direct Mail to the Right Server
Mail Exchange records tell the world which mail server is responsible for receiving email for your domain.

```
example.com. 3600 IN MX 10 mail.example.com.
```

This means for domain `example.com`, the primary mail exchanger is `mail.example.com` with priority 10. The lower the priority, the higher the preference.

### SPF - Sender Policy Framework
SPF is one of the primary email authentication methods. SPF allows the owner of the domain to specify which mail servers are authorized to send email on behalf of that domain.

```
example.com.   3600   IN   TXT   "v=spf1 ip4:abc.d.efg.h include:_spf.google.com -all"
```

This is a sample SPF record. This indicates that the domain authorizes a specific IPv4 address (abc.d.efg.h) and includes Google's mail servers as valid senders. `-all` means a hard fail, so if it fails, all emails are rejected automatically. `~all` means soft fail - if it fails, the mail gets marked as spam or suspicious.

### DKIM – DomainKeys Identified Mail
Another method of email authentication. DKIM ties the email to the domain's identity by using cryptographic signatures like RSA.

```
default._domainkey.example.com.  3600 IN TXT "v=DKIM1; k=rsa; p=MIIBIjANBgkqh...Ab (your public key) ...IDAQAB"
```

`p` is the public key, `v` is the version, and `k` is the key type.

### DMARC – Domain-Based Message Authentication, Reporting & Conformance

DMARC is another policy layer on top of SPF and DKIM. In simple terms, DMARC answers the question: *If an email claims to be from my domain, did it pass SPF and/or DKIM, and if not, what should the recipient server do with it?* When a receiver gets an email from `example.com`, it checks SPF and DKIM as usual.

```
_dmarc.example.com. 3600 IN TXT "v=DMARC1; p=quarantine; rua=mailto:[email protected]; ruf=mailto:[email protected]; fo=1"
```

Where `v` is the version, `rua` and `ruf` are the addresses where aggregate and forensic reports should be sent.

## To Sum Up
I tried to *deep summarize* the email protocols, DNS settings, and provide simple but deeper knowledge about emails. Thanks for reading, and if you have any suggestions or if I made any mistakes, feel free to reach out!

{% content-end %}

{% footer %}