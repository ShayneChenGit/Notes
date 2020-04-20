> Note:
>
> 1：生成管理员密码时，要用数字，不要用字母，不然后期会出错，我也不知道原因；
>
> 2：.ldif文件内，中间空行的那一行不要有空格；
>
> 3：运行base.ldif时，要输入之前所设定的数字密码，而不是生成的SSHA那一串。

参考链接

https://blog.csdn.net/weixin_41004350/article/details/89521170

https://cloud.tencent.com/developer/article/1490857

https://www.cnblogs.com/Mrhuangrui/p/8664363.html（access重要）



#### 1. 安装openldap

```
# yum 安装相关包
yum install -y openldap openldap-clients openldap-servers

# 复制一个默认配置到指定目录下,并授权，这一步一定要做，然后再启动服务，不然生产密码时会报错
cp /usr/share/openldap-servers/DB_CONFIG.example /var/lib/ldap/DB_CONFIG

# 授权给ldap用户,此用户yum安装时便会自动创建
chown -R ldap. /var/lib/ldap/DB_CONFIG

# 启动服务，先启动服务，配置后面再进行修改
systemctl start slapd
systemctl enable slapd

# 查看状态，正常启动则ok
systemctl status slapd
```

#### 2. 修改openldap配置

```
# 生成管理员密码,记录下这个密码，后面需要用到
slappasswd -s 123456
{SSHA}60Ses6pBE9WnViLDcLK/ILYlkv4TdLfI
```

##### 2.1 修改密码

1-changepwd.ldif

```
dn: olcDatabase={0}config,cn=config
changetype: modify
add: olcRootPW
olcRootPW: {SSHA}60Ses6pBE9WnViLDcLK/ILYlkv4TdLfI
```

```
ldapadd -Y EXTERNAL -H ldapi:/// -f 1-changepwd.ldif
```

##### 2.2 导入一些基本的 Schema

```
ldapadd -Y EXTERNAL -H ldapi:/// -f /etc/openldap/schema/cosine.ldif
ldapadd -Y EXTERNAL -H ldapi:/// -f /etc/openldap/schema/nis.ldif
ldapadd -Y EXTERNAL -H ldapi:/// -f /etc/openldap/schema/inetorgperson.ldif
ldapadd -Y EXTERNAL -H ldapi:/// -f /etc/openldap/schema/collective.ldif
ldapadd -Y EXTERNAL -H ldapi:/// -f /etc/openldap/schema/corba.ldif
ldapadd -Y EXTERNAL -H ldapi:/// -f /etc/openldap/schema/duaconf.ldif
ldapadd -Y EXTERNAL -H ldapi:/// -f /etc/openldap/schema/dyngroup.ldif
ldapadd -Y EXTERNAL -H ldapi:/// -f /etc/openldap/schema/java.ldif
ldapadd -Y EXTERNAL -H ldapi:/// -f /etc/openldap/schema/misc.ldif
ldapadd -Y EXTERNAL -H ldapi:/// -f /etc/openldap/schema/openldap.ldif
ldapadd -Y EXTERNAL -H ldapi:/// -f /etc/openldap/schema/pmi.ldif
ldapadd -Y EXTERNAL -H ldapi:/// -f /etc/openldap/schema/ppolicy.ldif
```

##### 2.3 修改domain

2-changedomain.ldif

```
dn: olcDatabase={1}monitor,cn=config
changetype: modify
replace: olcAccess
olcAccess: {0}to * by dn.base="gidNumber=0+uidNumber=0,cn=peercred,cn=external,cn=auth" read by dn.base="cn=admin,dc=demo,dc=com" read by * none

dn: olcDatabase={2}hdb,cn=config
changetype: modify
replace: olcSuffix
olcSuffix: dc=demo,dc=com

dn: olcDatabase={2}hdb,cn=config
changetype: modify
replace: olcRootDN
olcRootDN: cn=admin,dc=demo,dc=com

dn: olcDatabase={2}hdb,cn=config
changetype: modify
replace: olcRootPW
olcRootPW: {SSHA}60Ses6pBE9WnViLDcLK/ILYlkv4TdLfI

dn: olcDatabase={2}hdb,cn=config
changetype: modify
add: olcAccess
olcAccess: {0}to attrs=userPassword,shadowLastChange by dn="cn=admin,dc=demo,dc=com" write by anonymous auth by self write by * none
olcAccess: {1}to dn.base="" by * read
olcAccess: {2}to * by dn="cn=admin,dc=demo,dc=com" write by * read
```

```
ldapmodify -Y EXTERNAL -H ldapi:/// -f 2-changedomain.ldif
```

##### 2.4 验证

可以通过 search语法来确定账号密码是否正确

- ldapsearch 查询语法，     -H指定host， -D指定admin的账号，即rootdn, -w指定密码, -x启用认证

```
ldapsearch -H ldapi:/// -D "cn=admin,dc=demo,dc=com" -w 123456
```

##### 2.5 创建base组织

3-base.ldif

```
dn: dc=demo,dc=com
objectClass: top
objectClass: dcObject
objectClass: organization
o: Demo Company
dc: demo

dn: cn=admin,dc=demo,dc=com
objectClass: organizationalRole
cn: admin

dn: ou=People,dc=demo,dc=com
objectClass: organizationalUnit
ou: People

dn: ou=Group,dc=demo,dc=com
objectClass: organizationalRole
cn: Group
```

```
ldapadd -x -D cn=admin,dc=demo,dc=com -W -f 3-base.ldif
```

#### 3. 安装phpldapadmin

##### 3.1 安装

安装时可能会提示“No package phpldapadmin available”，则

```
yum install -y php-ldap php-mbstring php-pear php-xml
yum install -y epel-release
yum install -y phpldapadmin
```

##### 3.2 修改配置

修改apache的phpldapadmin配置文件

```
vim /etc/httpd/conf.d/phpldapadmin.conf
```

```
<IfModule mod_authz_core.c>
	# Apache 2.4
	Require all granted
</IfModule>
```

修改配置用DN登录ldap

```
vim /etc/phpldapadmin/config.php
```

```
# 398行，默认是使用uid进行登录，我这里改为cn，也就是用户名
$servers->setValue('login','attr','cn');
# 460行，关闭匿名登录，否则任何人都可以直接匿名登录查看所有人的信息
$servers->setValue('login','anon_bind',false);
# 519行，设置用户属性的唯一性，这里我将cn,sn加上了，以确保用户名的唯一性
$servers->setValue('unique','attrs',array('mail','uid','uidNumber','cn','sn'));
```

##### 3.3 启动apache

```
systemctl start httpd
systemctl enable httpd
```

404失败的原因

可能会访问 http://10.64.166.78/phpldapadmin/失败，404，是防火墙的原因

关闭防火墙

```
systemctl stop firewalld.service #停止防火墙服务
systemctl disable firewalld.service #禁用防火墙开机启动服务
```

或者添加防火墙允许

```
#添加389允许
firewall-cmd --add-service=ldap --permanent
firewall-cmd --reload

# 添加80允许
firewall-cmd --zone=public --add-port=80/tcp --permanent
```

出现success表明添加成功

##### 3.4 登录phpldapadmin界面

http://10.64.165.70/phpldapadmin/

#### 4. 新增user，用户等

##### 4.1 启用memberof功能

4-add_module_group.ldif

```
dn: cn=module,cn=config
cn: module
objectClass: olcModuleList
olcModulePath: /usr/lib64/openldap

dn: cn=module{0},cn=config
changetype: modify
add: olcModuleLoad
olcModuleLoad: memberof.la
```

```
ldapadd -Q -Y EXTERNAL -H ldapi:/// -f 4-add_module_group.ldif
```

5-add_group_objectClass.ldif

```
dn: olcOverlay=memberof,olcDatabase={2}hdb,cn=config
objectClass: olcConfig
objectClass: olcMemberOf
objectClass: olcOverlayConfig
objectClass: top
olcOverlay: memberof
olcMemberOfDangling: ignore
olcMemberOfRefInt: TRUE
olcMemberOfGroupOC: groupOfNames
olcMemberOfMemberAD: member
olcMemberOfMemberOfAD: memberOf
```

```
ldapadd -Q -Y EXTERNAL -H ldapi:/// -f 5-add_group_objectClass.ldif
```

（6，7 optional）

6-refint1.ldif

```
dn: cn=module{0},cn=config
add: olcmoduleload
olcmoduleload: refint
```

7-refint2.ldif

```
dn: olcOverlay=refint,olcDatabase={2}hdb,cn=config
objectClass: olcConfig
objectClass: olcOverlayConfig
objectClass: olcRefintConfig
objectClass: top
olcOverlay: refint
olcRefintAttribute: memberof uniqueMember  manager owner
```

```
ldapmodify -Q -Y EXTERNAL -H ldapi:/// -f 6-refint1.ldif
ldapadd -Q -Y EXTERNAL -H ldapi:/// -f 7-refint2.ldif
```

##### 4.2 新增user

###### 4.2.1 批量增加

8-adduser.ldif

```
dn: ou=研发部门,ou=People,dc=demo,dc=com
changetype: add
objectClass: organizationalUnit
ou: 研发部门

dn: ou=后台组,ou=研发部门,ou=People,dc=demo,dc=com
changetype: add
objectClass: organizationalUnit
ou: 后台组

dn: cn=ryan.miao,ou=后台组,ou=研发部门,ou=People,dc=demo,dc=com
changetype: add
objectClass: inetOrgPerson
cn: ryan.miao
departmentNumber: 1
sn: Miao
title: 大牛
mail: ryan.miao@demo.com
uid: 10000
displayName: 中文名

dn: cn=someone,ou=后台组,ou=研发部门,ou=People,dc=demo,dc=com
changetype: add
objectClass: inetOrgPerson
cn: someone
departmentNumber: 1
sn: someone
title: Java工程师
mail: someone@demo.com
uid: 10001
displayName: 某人

dn: ou=测试组,ou=研发部门,ou=People,dc=demo,dc=com
changetype: add
objectClass: organizationalUnit
ou: 测试组

dn: cn=tester.miao,ou=测试组,ou=研发部门,ou=People,dc=demo,dc=com
changetype: add
objectClass: inetOrgPerson
cn: tester.miao
departmentNumber: 2
sn: Miao
title: 测试工程师
mail: tester@demo.com
uid: 10002
displayName: 测试某人

dn: ou=HR,ou=People,dc=demo,dc=com
changetype: add
objectClass: organizationalUnit
ou: HR

dn: cn=fang.huang,ou=HR,ou=People,dc=demo,dc=com
changetype: add
objectClass: inetOrgPerson
cn: fang.huang
departmentNumber: 3
sn: Huang
title: HRBP
mail: fang.huang@demo.com
uid: 10003
displayName: 黄芳
```

```
ldapadd -x -D cn=admin,dc=demo,dc=com -w 123456 -f 8-adduser.ldif
```

###### 4.2.2 添加 Account 时指定密码

9-addone.ldif

```
dn: cn=hr-ryan,ou=HR,ou=People,dc=demo,dc=com
changetype: add
objectClass: inetOrgPerson
cn: hr-ryan
userPassword: 123456
departmentNumber: 3
sn: hr-ryan
title: HRBP
mail: hr-ryan@demo.com
uid: 10004
displayName: 我是猎头
```

```
ldapadd -x -D cn=admin,dc=demo,dc=com -w 123456 -f 9-addone.ldif
```

##### 4.3 新增group

10-addgroup.ldif

```
dn: cn=g-admin,ou=Group,dc=demo,dc=com
objectClass: groupOfNames
cn: g-admin
member: cn=ryan.miao,ou=后台组,ou=研发部门,ou=People,dc=demo,dc=com
member: cn=hr-ryan,ou=HR,ou=People,dc=demo,dc=com
```

```
ldapadd -x -D cn=admin,dc=demo,dc=com -w 123456 -f 10-addgroup.ldif
```

#### 5. phpldapadmin添加account

https://www.cnblogs.com/xiaomifeng0510/p/9564688.html

#### 6. 导入linux操作系统中的用户和组到openldap中

##### 6.1 创建账号

以备客户端测试登陆

```
useradd ldaptest01
passwd ldaptest01

useradd ldaptest02
passwd ldaptest02

useradd ldaptest03
passwd ldaptest03
```

至此，这些用户仅仅是系统上存在的用户（存储在/etc/passwd和/etc/shadow上），并没有在LDAP数据库里，所以要把这些用户导入到LDAP里面去。但LDAP只能识别特定格式的文件 即后缀为ldif的文件（也是文本文件），所以不能直接使用/etc/passwd和/etc/shadow。 需要migrationtools这个工具把这两个文件转变成LDAP能识别的文件。

##### 6.2 安装配置migrationtools

```
yum install migrationtools -y
```

##### 6.3 进入migrationtool配置目录

```
# cd /usr/share/migrationtools/
首先编辑migrate_common.ph

# vim migrate_common.ph
...
# Default DNS domain
$DEFAULT_MAIL_DOMAIN = "demo.com";


# Default base
$DEFAULT_BASE = "dc=demo,dc=com";
.......
保存退出。：-）
```

##### 6.4 生成ldif文件

下面利用pl脚本将/etc/passwd 和/etc/shadow生成LDAP能读懂的文件格式，保存在/tmp/下

```
./migrate_base.pl > /home/chenxin/base.ldif
./migrate_passwd.pl /etc/passwd > /home/chenxin/passwd.ldif 
./migrate_group.pl  /etc/group > /home/chenxin/group.ldif
```

##### 6.5 导入openldap

```
ldapadd -x -D "cn=admin,dc=demo,dc=com" -W -f /home/chenxin/base.ldif
ldapadd -x -D "cn=admin,dc=demo,dc=com" -W -f /home/chenxin/passwd.ldif
ldapadd -x -D "cn=admin,dc=demo,dc=com" -W -f /home/chenxin/group.ldif
```

过程若无报错，则LDAP服务端配置完毕

##### 6.6 重启slapd完成配置

```
service slapd restart
```

##### 6.7 现安装NFS，并把chenxin的家目录做NFS共享

```
# yum install nfs* -y
配置NFS共享：
# vi /etc/exports
--------------
/home/ldapuser1         *(rw,no_root_squash)
--------------
重启nfs服务：
# service rpcbind restart
# service nfs restart
```

PS.本地需要做ldap控制的账号都要导入到LDAP DB中，否则客户端配置无法正常识别登录。



#### 7. ppolicy overlay

[OpenLDAP Pasword policy (ppolicy)](onenote:#OpenLDAP Pasword policy (ppolicy)&section-id={BA89410E-4210-CC46-AF38-1DDE2E891324}&page-id={E76AE8A2-15FD-FC44-A943-891ED1613017}&end&base-path=https://d.docs.live.net/97e2ff3f4f2555a1/Documents/STUDY/LDAP.one)