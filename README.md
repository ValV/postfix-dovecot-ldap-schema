# Postfix-Dovecot LDAP schema

This schema includes attributes and object classes that are intended to satisfy all the LDAP lookups for Postfix and Dovecot for directory service based on OpenLDAP.

## Postfix section

This part is made of attributes:
- mailBox
- mailGroupMemberDN
- mailGroupMemberAddress

And object classes:
- postfixAccount
- postfixGroup
- postfixVirtualAccount
- postfixVirtualGroup
- postfixMailList

These attributes and object classes are based on the [Postfix LDAP README](http://www.postfix.org/LDAP_README.html) with more convenient (in my view) names.

> Note: some names may collide with other schemes, so before adding this one ensure that there's no other attributes with such names:
> `grep -Ri 'attributetype' /path/to/ldap | grep 'mail'`
> And no other object classes with such names:
> `grep -Ri 'objectclass' /path/to/ldap | grep 'postfix'`
> Where `/path/to/ldap` is the location of OpenLDAP configuration, e.g. in Debian it may be `/etc/ldap` and in Archlinux it may be `/etc/openldap`.

As mentioned above, all the attributes ans object classes are based on the official **Postfix LDAP README** but names differ. Here's the match-table between *official names* and the current.

| Postfix example        | Current scheme         |
| ---------------------- | ---------------------- |
| mailacceptinggeneralid | mail                   |
| maildrop               | mailBox                |
| memberdn               | mailGroupMemberDN      |
| memberaddr             | mailGroupMemberAddress |
| -                      | postfixAccount         |
| -                      | postfixGroup           |
| ldapuser               | postfixVirtualAccount  |
| ldapgroup              | postfixVirtualGroup    |
| maillist               | postfixMailList        |

> Note: *mail* attribute (from *core.schema*) is used for *mailAcceptingGeneralId*, so this schema does depend on *core.schema* (it seems to be ok since in most cases core schema is included by default).

Object classes *postfixAccount* and *postfixGroup* are auxiliary, so they are intended to be added to some objects, e.g. add *postfixAccount* to *posixAccount* to make **real user** account capable for Posfix LDAP lookups. That's why object classes for *ldapuser* and *ldapgroup* are called **virtual** - they can be declared as stand-alone (virtual) objects (users). Assume that one have **real users** in `ou=users,dc=example,dc=org` declared as *posixAccount* and augmented with *postfixAccount*, and **virtual users** in `ou=accounts,ou=email,dc=example,dc=org` declared as *postfixVirtualAccount*.

> The method above can be extrapolated to *postfixGroup* and *postfixVirtualGroup*

Object class postfixMailList can be used as in **Postfix LDAP README** example:
```
dn: cn=mailing-list-alpha, ou=lists, ou=email, dc=example, dc=org
cn: mailing-list-alpha
objectClass: postfixMailList
objectClass: top
mail: engineeringgroupalpha
mail: engineering-group-alpha
mailBox: alex-1964@example.org
mailBox: brigitte-1872@example.org
mailBox: max-1987@example.org
```

> Note: OID in this schema is of type *Experimental OpenLDAP* (as in [credativ/postfix-ldap-schema](https://github.com/credativ/postfix-ldap-schema)) - so to avoid collisions one should check before existing OIDs with `grep -Ri '1.3.6.1.4.1.4203.666' /path/to/ldap/`

## Dovecot section

This part relies solely on attributes, because of Dovecot's lookup specific. These attributes are used to store data for Dovecot's fields of **passdb** and **userdb** lookups.
According to the Dovecot's wiki on [Password Databases](http://wiki2.dovecot.org/PasswordDatabase#lookupdbs) fields that are necessary to return data from LDAP with **passdb** lookup:
- password (if one uses [Password Lookups](http://wiki2.dovecot.org/AuthDatabase/LDAP/PasswordLookups) over [Authentiaction Bind](http://wiki2.dovecot.org/AuthDatabase/LDAP/AuthBinds))
- user

Fields for **userdb** lookups as it is stated in [User Databases](http://wiki2.dovecot.org/UserDatabase) are as follows:
- uid
- gid
- home
- mail (overrides *mail_location* setting)
- quota_rule

And the [Extra Fields](http://wiki2.dovecot.org/PasswordDatabase/ExtraFields):
- user
- login_user
- allow_nets
- proxy
- proxy_maybe
- host
- nologin
- nodelay
- nopassword
- fail
- k5principals

Here's the match-table for Dovecot fields and the schema attributes.

| Dovecot field          | Schema attribute       |
| ---------------------- | ---------------------- |
| password               | mailPassword           |
| uid                    | mailUidNumber          |
| gid                    | mailGidNumber          |
| home                   | mailHomeDirectory      |
| mail                   | mailLocation           |
| quota_rule             | mailQuota              |
| user                   | -                      |
| login_user             | -                      |
| allow_nets             | -                      |
| proxy                  | -                      |
| proxy_maybe            | -                      |
| host                   | -                      |
| nologin                | -                      |
| nodelay                | -                      |
| nopassword             | mailNoPassword         |
| fail                   | mailDisabled           |
| k5principals           | -                      |

Attributes *mailPassword*, *mailUidNumber* and *mailGidNumber* are so-called **virtual** attributes, i.e. they're intended to be used for **virtual users**. For **real users** one should use *real passwords*, *real UIDs*, and *real GIDs* (e.g. *userPassword* from *core.schema*, *uidNumber*, and *gidNumber* from *nis.schema*).
Attributes *mailHomeDirectory* and *mailLocation* can be used with **real users**, if there's necessity to set up different location for mail for existing posix accounts.
Field *user* usually comes from *uid* (also *userid*, *core.schema*).

##### TODO
- [ ] add the rest of the attributes for Dovecot fields
- [ ] check *uid*s or *userid*s for *postfixVirtualAccount*

