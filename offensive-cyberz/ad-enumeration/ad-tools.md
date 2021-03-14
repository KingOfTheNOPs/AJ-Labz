---
description: Just some neat tools
---

# AD Tools

### Windapsearch

`Windapsearch` is a Python script used to perform anonymous and authenticated LDAP enumeration of AD users, groups, and computers using LDAP queries. It is an alternative to tools such as `ldapsearch`, which require you to craft custom LDAP queries. We can use it to confirm LDAP NULL session authentication but providing a blank username with `-u ""` and add`--functionality` to confirm the domain functional level.

[https://github.com/ropnop/windapsearch](https://github.com/ropnop/windapsearch)

