1、以root用户登录Mysql
```bash
mysql -uroot -proot
# 或
sudo mysql
```
2、切换到mysql数据库
```
use mysql
```
3、添加用户
```sql
//只允许指定ip连接
create user '新用户名'@'localhost' identified by '密码';
//允许所有ip连接（用通配符%表示）
create user '新用户名'@'%' identified by '密码';
```
4、为新用户授权
```sql
//基本格式如下
grant all privileges on 数据库名.表名 to '新用户名'@'指定ip';
//示例
//允许访问所有数据库下的所有表
grant all privileges on *.* to '新用户名'@'指定ip';
//指定数据库下的指定表
grant all privileges on test.test to '新用户名'@'指定ip';
```
5、设置用户操作权限
```sql
//设置用户拥有所有权限也就是管理员
grant all privileges on *.* to '新用户名'@'指定ip' identified by '新用户密码' WITH GRANT OPTION;
//拥有查询权限
grant select on *.* to '新用户名'@'指定ip' identified by '新用户密码' WITH GRANT OPTION;
//其它操作权限说明,select查询 insert插入 delete删除 update修改

//取消用户查询的查询权限
REVOKE select ON what FROM '新用户名';
```
6、删除用户
```sql
DROP USER username@localhost;
```
7、修改后刷新权限
```sql
FLUSH PRIVILEGES;
```


