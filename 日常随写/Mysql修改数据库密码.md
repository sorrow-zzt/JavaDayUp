## Mysql修改数据库密码

### 1.进入skip-grant-tables模式启动

**skip-grant-tables表示，在启动mysql时不启动授权表功能，可以直接免密码登录**

```sql
#修改/etc/my.cnf文件
vim /etc/my.cnf

#在[mysqld]区域添加配置,并保存my.cnf文件
skip-grant-tables

#重启mysql
systemctl restart mysqld

#登录mysql
mysql -u root -p

#如果出现输入密码，直接回车，就可以进入数据库了
```

### 2.1进入数据库修改密码(MySQL5.7)

#### 2.1.1修改root密码

```sql
#登录mysql，此时还没有进入数据库，使用如下命令
use mysql;

#修改root密码（mysql5.7版本）
update user set authentication_string = password('新密码'), password_expired = 'N',password_last_changed = now() where user = 'root';

#如果你的mysql是5.6版本修改root密码（mysql5.6版本）
update user set password=password('新密码') where user='root';

#使其生效
flush privileges;

#退出
exit;
```

#### 2.1.2新增管理员用户

如果你不想修改root密码，可以新增一个管理员用户，操作如下：

```sql
#登录mysql，此时还没有进入数据库，使用如下命令
use mysql;

#刷新数据库
flush privileges;

#创建一个用户，并赋予管理员权限
grant all privileges on *.* to '用户'@'%' identified by '密码';

#例如，创建一个admin用户，密码为admin
grant all privileges on *.* to 'admin'@'%' identified by 'admin';
```

#### 2.1.3重启服务器

上面操作完成之后，其实还没有完，需要关闭授权表功能，重启服务器:

```sql
#修改/etc/my.cnf文件
vim /etc/my.cnf

#在[mysqld]区域删除改配置,并保存my.cnf文件
#skip-grant-tables

#重启mysql
systemctl restart mysqld

#此时，修改完毕
```



### 2.2进入数据库修改密码(MySQL8.0)

```sql
#登录mysql，此时还没有进入数据库，使用如下命令
use mysql;

#在skip-grant-tables模式下，将root密码置空
update user set authentication_string = '' where user = 'root';

#退出，将/etc/my.cnf文件下的skip-grant-tables去掉，重启服务器(不确定是否必须)
reboot

#登录mysql
mysql -u root -p

#因为密码置空，直接回车，进入数据库之后，修改密码
ALTER USER 'root'@'%' IDENTIFIED BY 'Hello@123456';

#因为mysql8，使用强校验，所以，如果密码过于简单，会报错，密码尽量搞复杂些！
```

## 3.mysql删库回复简介

```
找到/etc/my.cnf并编辑（没有my.cnf的时候就找my.ini）；添加
 log-bin=mysql-bin 
 expire_logs_days=7(日志保留天数)
然后重启mysql
```

