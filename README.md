# Postfix-Dovecot LDAP schema

This schema includes attributes and object classes that are intended to satisfy all the LDAP lookups for Postfix and Dovecot for directory service based on OpenLDAP.

## Postfix section

This part is made of attributes:
- mailBox
- mailGroupMemberDN
- mailGroupMemberAddress

And object classes
- postfixAccount
- postfixGroup
- postfixVirtualAccount
- postfixVirtualGroup
- postfixMailList

These attributes and object classes are based on the [Postfix LDAP README](http://www.postfix.org/LDAP_README.html) with more convenient (from my view) names.

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
