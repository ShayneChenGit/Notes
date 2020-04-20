#### 1. 加载 ppolicy schema

```
ldapadd -Q -Y EXTERNAL -H ldapi:/// -f /etc/openldap/schema/ppolicy.ldif
```

加完了再查看下 schema 列表，已经加上了:

```
ldapsearch -Q -LLL -Y EXTERNAL -H ldapi:/// -b cn=schema,cn=config dn
```

<img src="https://raw.githubusercontent.com/ShayneChenGit/ImageHosting/master/img/image-20200420231729777.png" alt="image-20200420231729777" style="zoom:50%;" />

#### 2. 加载 ppolicy module

```
vim ppolicy_module.ldif
----------------------------------------------------------------
dn:cn=module{0},cn=config
changetype: modify
add: olcModuleLoad
olcModuleLoad: ppolicy
```

```
ldapadd -Y EXTERNAL -H ldapi:/// -f ppolicy_module.ldif
```

查看 module 是否加载好了：

```
ldapsearch -Q -LLL -Y EXTERNAL -H ldapi:/// -b cn=module{0},cn=config
```

<img src="https://raw.githubusercontent.com/ShayneChenGit/ImageHosting/master/img/image-20200420231833931.png" alt="image-20200420231833931" style="zoom:50%;" />

#### 3. 加载 ppolicy overlay

```
vim ppolicy_overlay.ldif
----------------------------------------------------------------
dn: olcOverlay=ppolicy,olcDatabase={2}hdb,cn=config
changetype: add
objectClass: olcOverlayConfig
objectClass: olcPPolicyConfig
olcOverlay: ppolicy
olcPPolicyDefault: cn=ppolicy,ou=policies,dc=demo,dc=com
olcPPolicyHashCleartext: TRUE
olcPPolicyUseLockout: TRUE
```

```
ldapadd -YEXTERNAL -H ldapi:/// -f ./ppolicy_overlay.ldif
```

查看是否添加成功

```
vim /etc/openldap/slapd.d/cn\=config/olcDatabase\=\{2\}hdb/olcOverlay\=\{1\}ppolicy.ldif
```

<img src="https://raw.githubusercontent.com/ShayneChenGit/ImageHosting/master/img/image-20200420231951742.png" alt="image-20200420231951742" style="zoom:50%;" />

#### 4. 配置 default PPolicy 和规则

```
vi default_ppolicy.ldif
----------------------------------------------------------------
dn: ou=policies,dc=demo,dc=com
objectClass: organizationalUnit
objectClass: top
ou: policies

dn: cn=default,ou=policies,dc=demo,dc=com
cn: default
objectClass: pwdPolicy
objectClass: person
objectClass: top
pwdAttribute: userPassword
pwdMinAge: 0
pwdMaxAge: 7776000
pwdInHistory: 5
pwdCheckQuality: 0
pwdMinLength: 5
pwdExpireWarning: 6480000
pwdGraceAuthNLimit: 5
pwdLockout: TRUE
pwdLockoutDuration: 300000
pwdMaxFailure: 5
pwdFailureCountInterval: 30
pwdMustChange: FALSE
pwdAllowUserChange: TRUE
pwdSafeModify: FALSE
sn: dummy value
```

<img src="https://raw.githubusercontent.com/ShayneChenGit/ImageHosting/master/img/image-20200420232036959.png" alt="image-20200420232036959" style="zoom:50%;" />

可以使用ldapwhoami命令来输入错误密码，制造登陆失败

```
ldapwhoami -H ldap://192.168.141.136 -x -D "uid=chenxin,ou=People,dc=demo,dn=com" -w shay2000
```

```
ldapsearch -H ldap://192.168.141.136 -x -LLL uid=chenxin +
```

![image-20200420232120892](https://raw.githubusercontent.com/ShayneChenGit/ImageHosting/master/img/image-20200420232120892.png)

https://www.openldap.org/lists/openldap-technical/201111/msg00165.html

filter=(&(objectClass=inetOrgPerson)(!(organizationalStatus=0)))

(&(objectClass=*)(!(organizationalStatus=0))) 

(&(objectClass=*)(!(pwdAccountLockedTime=000001010000Z)))

 https://www.openldap.org/lists/openldap-technical/201504/msg00139.html