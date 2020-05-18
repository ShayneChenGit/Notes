#### windows

Win10截图快捷键：shift + windows + s

##### nslookup

###### 1. 查询domain的MX记录

nslookup -query=mx 'sina.net'

###### 2. 根据ip查hostname

```
nbtstat -a 10.64.166.26
```



#### Linux

##### vim

显示行号：esc -> :set nu

跳转到指定行：esc -> :行号

删除整行：dd

删除多行： esc -> :32,65d

全选：ggvG

复制选中：y

复制整行：yy

粘贴：p

撤销：u

##### 进程

进程筛选：

```shell
ps -ef | grep ldap_service_daemon.py
```

进程筛选且不包含grep本身： 

```shell
ps -ef | grep ldap_service_daemon.py | grep -v grep
```

显示进程筛选且不包含grep本身的pid：

```shell
ps -ef | grep ldap_service_daemon.py | grep -v grep | awk {'print $2'}
```





##### 查找

grep -r 'search_string' ./

find ./ -name 'localclient*'



#### Mac OS



