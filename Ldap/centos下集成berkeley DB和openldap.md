操作系统：centos 7

openldap版本：

Berkeley DB版本：5.1.29

参考链接：

https://blog.csdn.net/myhes/article/details/97972841

https://www.cnblogs.com/Mrhuangrui/p/8664363.html

#### 安装DB

##### 1. 下载

```
cd /usr/local/src
wget https://download.oracle.com/berkeley-db/db-5.1.29.tar.gz
tar -zxvf db-5.1.29.tar.gz
cd db-5.1.29/build_unix/
../dist/configure --prefix=/usr/local/berkeleydb-5.1.29
make
make install
```

##### 2. 解压

```
tar -zxvf db-5.1.29.tar.gz
```

##### 3. 进入编译目录

```
cd db-5.1.29/build_unix/
```

##### 4. 配置

```
../dist/configure --prefix=/usr/local/berkeleydb-5.1.29
```

##### 5. 编译

```
make
make install
```

##### 6. 查看是否安装成功

```
ls /usr/local/berkeleydb-5.1.29/
```

##### 7. 库文件连接创建

```
echo “/usr/local/berkeleydb-5.1.29/lib/” > /etc/ld.so.conf
```

##### 8. 配置查看

```
ldconfig -v
```

#### 安装openldap

```
tar -zxvf openldap-2.4.48.tgz

yum install *ltdl*

export CPPFLAGS="-I/usr/local/berkeleydb-5.1.29/include" 
export LDFLAGS="-L/usr/local/berkeleydb-5.1.29/lib" 
export LD_LIBRARY_PATH="/usr/local/ssl/lib:/usr/local/berkeleydb-5.1.29/lib" 

./configure --prefix=/usr/local/openldap-2.4.48 --enable-syslog --enable-modules --enable-debug --with-tls CPPFLAGS=-I/usr/local/berkeleydb-5.1.29/include/ LDFLAGS=-L/usr/local/berkeleydb-5.1.29/lib/

make depend
make
make install
```



