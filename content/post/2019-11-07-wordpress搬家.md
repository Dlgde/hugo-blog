---
title: WordPress搬家
author: dlgde
type: post
date: 2019-11-07T03:13:45+00:00
url: /archives/177
views:
  - 381
categories:
  - web

---
# WordPress搬家

### 主要采坑过程如下：

  1. 复制之前的php目录到目标新机器
  2. 在新机器上面安装openresty(nginx),
  3. apt install php7.2-fpm php7.2-mysql php7.2-curl php7.2-json php7.2-mbstring php7.2-xml php7.2-intl,安装php服务
  4. apt-get install mysql-server 安装数据库
  5. 查看php服务，service php7.2-fpm status
  6. 目录下/etc/php/7.2/fpm/pool.d，www.conf中将listen改为9000
  7. mysql轻量数据库迁移： 
      * **导出数据**：`mysqldump -u root -p wordpress >dumpout.sql`,并将导出的copy到目标机器
      * **导入数据**：先创建数据库(`create database worpress`),执行`mysql -u root -p wordpress < dumpout.sql`
  8. 修改WordPress配置文件，`/data/www/wordpress/wp-config.php`中的数据库配置</p> 
  9. 重新启动php服务

 10. 修改域名解析，至此整个过程完成

 11. mysql赋予权限：`grant 权限 on 数据库.* to 用户名@登录主机 identified by "密码"`

 12. 修改密码：
    
      * `alter user user() identified by '*****';`alter user root@localhost identified by '\****'
    
      * `update mysql.user set authentication_string=password('*****') where user='root' and host='localhost';`