---
layout: postlayout
title: 织梦（DEDECMS）网站程序及数据库迁移搬家
categories: [seo]
tags: [seo,web]
---
织梦（DEDECMS）网站程序及数据库迁移搬家

首先我们将目的地的数据库账号密码获取到，并且保证持有一个数据库或拥有创建数据库的权限。

1、首先登录织梦的后台，进入到“系统》数据库备份/还原”栏目。

2、如果之前有备份过数据，就需要将织梦程序根目录下的data文件夹下的backupdata文件夹改名为其他名称，如"_backupdata"。

3、全选所有的表，根据情况选择MYSQL版本，按照默认选项提交备份。

4、备份成功后，将网站所有程序和目录上传到新的服务器。(主要文件有backupdata）

5、然后将已上传程序根目录中install文件夹下的“index.php.bak”改名为“index.php”，并且将“module-install.php.bak”更改为“module-install.php”，删除“install_lock.txt”文件。

6、在浏览器地址栏输入“你的域名/install/index.php”开始正常安装程序，其中数据库账号密码需填写目的地服务器的资料，网站程序的账号密码保持默认。

7、登录网站程序后台“http://域名/后台路径”，进入到“系统》数据库备份/还原》右上角数据还原》左下角开始还原数据”。

8、当数据完整的还原后，把网站程序中的install文件夹权限设置为不可读或直接删掉。
