```python
import ldap

ldapconn = ldap.initialize('ldap://10.64.166.78')
ldapconn.simple_bind_s('cn=admin,dc=yaobili,dc=com', '123456')
searchScope = ldap.SCOPE_SUBTREE
searchFilter = 'cn=admin'
base_dn = 'dc=yaobili,dc=com'
print ldapconn.search_s(base_dn, searchScope, searchFilter, None)
```

