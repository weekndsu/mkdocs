# Harbor API

# 1       项目管理

## 1.1     查看仓库中项目详细信息

```
curl -u "admin:Harbor12345" -X GET -H "Content-Type: application/json" "https://192.168.56.106/api/projects/{project_id}"

curl -u "admin:Harbor12345" -X GET -H "Content-Type: application/json" "https://192.168.56.106/api/projects?project_name=guest"
```

## 1.2     搜索镜像

```
curl  -u "admin:Harbor12345"  -X GET -H "Content-Type: application/json" "https://192.168.56.106/api/search?q=nginx"
```

## 1.3     删除项目

```
curl  -u "admin:Harbor12345"  -X DELETE  -H "Content-Type: application/json" "https://192.168.56.106/api/projects/{project_id}"
```

## 1.4     创建项目

```
curl -u "admin:Harbor12345" -X POST -H "Content-Type: application/json" "https://192.168.56.106/api/projects" -d @createproject.json

 

createproject.json例子

{

  "project_name": "testrpo",

  "public": 0

}
```

## 1.5     查看项目日志

```
curl -u "admin:Harbor12345" -X POST -H "Content-Type: application/json" "https://192.168.56.106/api/projects/{project_id}/logs/filter" -d @log.json

 

[root@dcos-hub json]# cat log.json

{

  "username": "admin"

}
```

# 2       账号管理

## 2.1     创建账号

```
curl -u "admin:Harbor12345" -X POST -H "Content-Type: application/json" "https://192.168.56.106/api/users" -d @user.json

 

[root@dcos-hub json]# cat >user.json

{

  "user_id": 5,

  "username": "xinju",

  "email": "xinju@gmail.com",

  "password": "Xinju12345",

  "realname": "xinju",

  "role_id": 2

}
```

## 2.2     获取用户信息

```
curl -u "admin:Harbor12345" -X GET -H "Content-Type: application/json" "https://192.168.56.106/api/users"
```

2.3     获取当前用户信息 

```
curl -u "admin:Harbor12345" -X GET -H "Content-Type: application/json" "https://192.168.56.106/api/users/current"
```

## 2.4     删除用户

```
curl -u "admin:Harbor12345" -X DELETE  -H "Content-Type: application/json" "https://192.168.56.106/api/users/{user_id}"
```

## 2.5     修改用户密码

```
curl -u "admin:Harbor12345" -X PUT -H "Content-Type: application/json" "https://192.168.56.106/api/users/{user_id}/password" -d @uppwd.json

 

[root@dcos-hub json]# cat uppwd.json

{
  "old_password": "Harbor123456",
  "new_password": "Harbor12345"
}
```

# 3       用户权限管理

## 3.1     查看项目相关角色

```
curl -u "admin:Harbor12345" -X GET -H "Content-Type: application/json" "https://192.168.56.106/api/projects/{project_id}/members/"
```

## 3.2     项目添加角色

```
curl -u "jaymarco:Harbor123456" -X POST  -H "Content-Type: application/json" "https://192.168.56.106/api/projects/{project_id}/members/" -d @role.json

 

[root@dcos-hub json]# cat role.json

{

  "roles": [

    3

  ],

  "username": "guest"

}
```

 用jaymarco用户创建一个snc_dcos项目，并对snc_dcos加一个权限

```
curl -u "jaymarco:Harbor123456" -X POST -H "Content-Type: application/json" "https://192.168.56.106/api/projects" -d @createproject.json
```

## 3.3     删除项目中用户权限

```
curl -u "admin:Harbor12345" -X DELETE -H "Content-Type: application/json" "https://192.168.56.106/api/projects/{project_id}/members/{user_id}"
```



## 3.4     获取与用户相关的项目编号和存储库编号

```
curl -u "admin:Harbor12345" -X GET -H "Content-Type: application/json" "https://192.168.56.106/api/statistics"
```



## 3.5     修改当前用户角色

has_admin_role ：0  普通用户

has_admin_role ：1  管理员

```
curl -u "admin:Harbor12345" -X PUT -H "Content-Type: application/json" "https://192.168.56.106/api/users/{user_id}/sysadmin " -d @chgrole.json

[root@dcos-hub json]# cat >chgrole.json

{

  "has_admin_role": 1

}
```

# 4       镜像管理

## 4.1     查询镜像

```
curl -u "admin:Harbor12345" -X GET -H "Content-Type: application/json" "https://192.168.56.106/api/repositories?project_id={project_id}&q=dcos%2Fcentos"
```

 

## 4.2    删除镜像

```
curl -u "admin:Harbor12345" -X DELETE -H "Content-Type: application/json" "https://192.168.56.106/api/repositories?repo_name=dcos%2Fetcd "
```

## 4.3     获取镜像标签

```
curl -u "admin:Harbor12345" -X GET -H "Content-Type: application/json" "https://192.168.56.106/api/repositories/tags?repo_name=dcos%2Fcentos"
```