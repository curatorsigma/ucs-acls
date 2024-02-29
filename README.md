# General Notes
This repo is a snippet dump. No development in this repo!

# What is this?
A snippet of tasks in ansible and the associated templates that write the following ACLs to an existing UCS

```slapd.conf
# block general read access. Because several objects need read acess which is not specifically allowed
# we take the safe route and blacklist users (and self registered users) from read

# NB:
# 1. compare and search are not used otherwise.
# 2. auth is only granted to anonymous (thus, dn.subtree=... does not apply to them)
# 1+2 means that we can disclose stop (or none stop if you prefer). There is no AC later that would need to set other access levels
# for the users matching here

access to *
    # I explicitly break this because we later need to set read access for this socket
    by sockname="PATH=/var/run/slapd/ldapi" break
    # We want Admins to be able to read. we break so that write access can also be defined later (UCS handles this)
    by group/univentionGroup/uniqueMember="cn=Domain Admins,cn=groups,dc=jgdresden,dc=intranet" read break
    by dn.subtree="cn=users,dc=jgdresden,dc=intranet" disclose stop
    by dn.subtree="cn=self registered users,dc=jgdresden,dc=intranet" disclose stop
```

