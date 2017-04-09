# Base mail server role
This role configures postfix and dovecot to provide a basic mailserver with SMTP, IMAP and (ugh.) POP3

## How to use
Look into `defaults/main.yml` for settings to change. You also need a working LDAP and a MariaDB/MySQL database with a schema similar to `mail_mock`. This role also depends on a validating resolver, so you should probably use `dns-resolver`

## Postfix
Postfix does users for the main domain directly against LDAP. Aliases are resolved via MariaDB. For the stored procedures, look into `mail_mock`. Authentication is done via SASL (Postfix can't do this via LDAP) and mail is delivered via LMTP. 
### Configuration specifics:
* Users need to authenticate to be able to send mail
* Authenticated users can only use their own adress and aliases which have the `can_send` flag set to 1 as a source adress. If you test this: Changing the from field in Thunderbird's compose view does not actually change the field, so don't get confused if it doesn't seem to work. You have to actually change the account's adress in the account settings view.
* TLS is required for clients. A self-signed certificate is copied iff no certificate is found

## Dovecot
Dovecot does auth using LDAP and additionally provides a SASL socket. Both sockets are created in the postfix chroot. POP3 does not actually delete mails, but only marks them as read and tags them with a special tag. All mails that are tagged like this are invisible on the next fetch, allowing people to use the same account with both POP3 and IMAP without loosing mails.

### Integration with antispam
If the antispam role is set up, Dovecot will create a virtual mailbox called '__spamuser_[spam, ham]' that all tagged messages will get forwarded to. This is necessary because the old antispam plugin does not work reliably and directly invoking sa-learn from sieve means that moving a mail in the web interface takes up to 5 seconds. Also, it makes it hard to actually sync the spamfilter state.

### Replication
If there is more than a single server in the group, replication between them will be automatically enabled. This will use SSH (less dependencies than SSL, for details see the `30-replication.conf.j2`), please make sure that the user is allowed to use SSH if you have custom restrictions in place!

# TODO
## probably in separate roles (which this role maybe gets to depend on)
* Mailman (ideally mailman3) and dependencies
