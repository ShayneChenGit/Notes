ldapsearch 查询语法， -H指定host， -D指定admin的账号，即rootdn, -w指定密码, -x启用认证 filter 返回哪些值

```
ldapsearch -H ldapi:/// -D "cn=admin,dc=demo,dc=com" -w 123456
```

```
ldapsearch -x -D "cn=admin,dc=demo,dc=com" -w 123456 -b dc=demo,dc=com
```

```
ldapsearch -x -h 10.64.166.78 -p 389 -b dc=demo,dc=com -D "cn=admin,dc=demo,dc=com" -w 123456
```

```
# filter = "(&(mail=ziye@demo)(uid=zdeng))"
# 返回值： mail
ldapsearch -x -h 10.64.166.78 -p 389 -b dc=demo,dc=com -D "cn=admin,dc=demo,dc=com" -w 123456 "(&(mail=ziye@demo)(uid=zdeng))" mail
```

> Note:
>
> 1：生成管理员密码时，要用数字，不要用字母，不然后期会出错，我也不知道原因；
>
> 2：.ldif文件内，中间空行的那一行不要有空格；
>
> 3：运行base.ldif时，要输入之前所设定的数字密码，而不是生成的SSHA那一串。