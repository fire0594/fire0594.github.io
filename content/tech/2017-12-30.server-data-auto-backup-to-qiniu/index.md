---
slug: "server-data-auto-backup-to-qiniu"
title: "将网站文件和数据库自动备份到七牛云端"
keywords: "服务器,python,七牛"
description: "手动备份网站数据太麻烦，用python弄个小工具自动加密压缩后上传到七牛。"
date: 2017-12-30T15:04:36+08:00
draft: false
---

手动备份网站数据太麻烦，弄了个python小工具
功能很简单，就是将导出的网站数据库和网站文件一起加密压缩后上传到七牛

用的是七牛的SDK，所以要先 pip install qiniu 安装下七牛的Python SDK
压缩包是调用系统的7z命令，所以要先 apt-get install p7zip-full 安装7z

需要的配置项
七牛：AccessKey、SecretKey、Bucket名称
数据库：主机、用户名、密码、编码、要备份的数据库名称（列表）
网站文件（列表）
备份完后是否清除本地文件
压缩文件密码

python代码：

```python
#!/usr/bin/python
#-*- coding:utf-8 -*-

import os,time
from qiniu import Auth, put_file, etag, urlsafe_base64_encode
import qiniu.config

"""七牛云配置"""
AccessKey = '七牛的AccessKey'
SecretKey = '七牛的SecretKey'
BucketName = '七牛的存储空间名称'

"""mysql配置"""
db_host = "localhost"
db_user = "数据库用户名"
db_passwd = "数据库密码"
db_charset = "utf8"
db_names = ["database1", "database2"]

"""网站目录配置"""
html_paths = ["/var/www/html1", "/var/www/html2"]

"""是否清除本地文件"""
remove_local = "yes"
zip_passwd = "123123"

zip_file = "Backup_" + time.strftime("%Y%m%d%H%M%S") + ".zip"
for db_name in db_names:
  sql_file = "/tmp/" + db_name + ".sql"
  os.system("mysqldump -h%s -u%s -p%s %s --default_character-set=%s > %s" %(db_host, db_user, db_passwd, db_name, db_charset, sql_file))
  os.system("7z a -p%s -y %s %s" %(zip_passwd, zip_file, sql_file))
  if (remove_local == "yes"):
    os.remove(sql_file)
for html_path in html_paths:
  os.system("7z a -p%s -y %s %s" %(zip_passwd, zip_file, html_path))
q = Auth(AccessKey, SecretKey)
token = q.upload_token(BucketName, os.path.basename(zip_file), 3600)
ret, info = put_file(token, os.path.basename(zip_file), zip_file)
print(info)
if (remove_local == "yes"):
  os.remove(zip_file)
```
