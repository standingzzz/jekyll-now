---
layout: post
title: Oracle12c Install to Centos
categories: Tec
tags: Oracle Database
date: 2017-08-15
---

### JRE required for runInstaller.

```
sudo rpm -ivh jre-8u144-linux-x64.rpm
```
- - -
### for garbled characters.

```
./runInstaller -jreLoc /usr/lib/jvm/java-8-oracle/jre/
```

- - -
### if unzip error.

```
cp /usr/bin/unzip ./install/unzip
```

# Connect Test
### if ora-12541 tns no listener, maybe.

##### try telnet

```
$ telnet 192.168.100.1 1521
Trying 192.168.100.1...
telnet: connect to address 192.168.100.1: Connection refused
telnet: Unable to connect to remote host
```

##### port authorization

```
firewall-cmd --zone=public --add-port=1521/tcp
firewall-cmd --zone=public --add-port=1521/tcp --permanent
firewall-cmd –reload
firewall-cmd --list-all

  ports: 1521/tcp
```

##### retry

```
$ telnet 192.168.100.1 1521
Trying 192.168.100.1...
Connected to oracle12c.test.
```

##### connect

```
SQL> conn TEST01@ORA12PDB
Enter password: 
Connected.
```

```
ORA12PDB=
  (DESCRIPTION =
    (ADDRESS = (PROTOCOL = TCP)(HOST = 192.168.100.1)(PORT = 1521))
    (CONNECT_DATA =
      (SERVER = DEDICATED)
      (SERVICE_NAME = pdb.localdomain)
    )
  )
```
