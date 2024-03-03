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
    # do not change permissions for self
    by self break
    # Domain Admins get no special read access elsewhere, we have to add this manually
    # we break, because there are special write-access rules for parts of the LDAP tree
    # that we still need to parse (they come after this snippet)
    by group/univentionGroup/uniqueMember="cn=Domain Admins,cn=groups,LDAP_BASE" read break
    # these are the read accesses we have to override
    by dn.subtree="cn=users,dc=jgdresden,dc=intranet" disclose stop
    by dn.subtree="cn=self registered users,LDAP_BASE" disclose stop
    # need to continue parsing for auth access (by anonymous auth comes later)
    by * break
```

# How do I do this manually?
1. Write the ACL file (like the above snippet) into
`/etc/univention/templates/files/etc/ldap/slapd.conf.d/51user_defined.acl`
Important is the directory and that the file starts with a number in range(50,60), because ACL order matters.
2. Write the following block (or multiple, if you have multiple userdefined ACL files) to `/etc/univention/templates/info/univention-ldap-server.info` so that it is in the correct position according to file name numbers.
```
Type: subfile
Multifile: etc/ldap/slapd.conf
Subfile: etc/ldap/slapd.conf.d/51user_defined.acl
```

3. `ucr commit /etc/ldap/slapd.conf`
    - NB: If your ACL does not have correct slapd.conf Syntax, this will throw a unicode error (for some reason)
4. make sure the ACL is now written to `/etc/ldap/slapd.conf`
5. `service slapd restart`

