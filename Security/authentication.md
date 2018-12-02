# 认证

###### MongoDB权限认证说明：
1. MongoDB安装后默认不开权限认证
2. MongoDB分为管理员数据库（admin）和普通数据库（自己命名，通过use命令）
3. MongoDB默认没有超级管理员账号，所以要先添加超级管理员账号，再开启权限认证。
```python
> use admin 
> show collections
> db.createUser(
   {
     user: "admin",
     pwd: "xxxxxxxxxxxxxxx",
     roles: [ { role: "userAdminAnyDatabase", db: "admin" } ]
   }
)
```
4.开启权限认证
```python
4.1.sudo vim /etc/mongod.conf

4.2.取消security的注释，加上 authorization: enabled
security:
  authorization: enabled

4.3.重启服务：systemctl restart mongod.service
```

5. 管理admin数据库的用户只能在admin里面创建；管理其它数据库的用户既可以在admin数据库创建也可以在本身数据库创建。

在admin创建的管理stu_manage的用户
 ```python
> db.auth('admin', 'admin')
1
> db.createUser({
... ... user: 'hongxin',
... ... pwd: 'hongxin',
... ... roles: [ { role: "dbOwner", db: "stu_manage"}]
... ... })
Successfully added user: {
	"user" : "hongxin",
	"roles" : [
		{
			"role" : "dbOwner",
			"db" : "stu_manage"
		}
	]
}
```

在stu_manage创建的管理stu_manage的用户
```python
> use stu_manage
switched to db stu_manage
> db.createUser({
... ... ... user: 'hongxin',
... ... ... pwd: 'hongxin',
... ... ... roles: [ { role: "dbOwner", db: "stu_manage"}]
... ... ... })
Successfully added user: {
	"user" : "hongxin",
	"roles" : [
		{
			"role" : "dbOwner",
			"db" : "stu_manage"
		}
	]
}
```

比较：
1.账户密码及角色可一致（红色划线），类似python的包名
2.可通过 show users 查看用户有哪个数据库创建
3. admin创建的，需在admin进行验证后，再切换到stu_manage才能进行操作；stu_manage创建的,直接在 stu_manage 验证后即可操作.

##### 基本使用（命令行）：
https://www.jianshu.com/p/a4e94bb8a052

##### 基本使用（Python）：
Python操作我用的是Pycharm，Pycharm里面有个查看数据的插件，叫 "Mongo Explorer", 个人用起来还可以。

新建连接（只标出重点）：
label: 链接名
User Database：数据库名
Authentication - Auth.database： 在哪个数据库创建的用户
Authentication - Auth.mechanism： 认证的协议，可通过 show users 查看
![image.png](https://upload-images.jianshu.io/upload_images/5034811-1308c573e6ebc2c0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



参考：
[如何对MongoDB 3.2.7进行用户权限管理配置](https://www.jianshu.com/p/a4e94bb8a052)
