# 📦 从零到一：手把手教你将本地Spring Boot + Vue项目部署到阿里云ECS

> 本文记录了将“智慧农业管理系统”从本地代码到线上可访问的全过程，包含服务器购买、环境配置、项目打包、部署上线等完整步骤，适合新手参考。

------

## 📌 目录

1. 前言
2. 第一步：购买云服务器
3. 第二步：连接服务器
4. 第三步：安装运行环境
5. 第四步：配置数据库
6. 第五步：打包后端项目
7. 第六步：打包前端项目
8. 第七步：上传并启动后端
9. 第八步：部署前端到Nginx
10. 第九步：配置Nginx反向代理
11. 第十步：开放端口与安全组
12. 第十一步：测试访问
13. 常见问题解决
14. 总结

------

## 前言

你是否也遇到过这样的问题：在本地辛辛苦苦写好的项目，想让朋友看到却只能发个本地 `localhost` 截图？其实把项目部署到云服务器上并不难，本文将带你一步步完成部署。

**项目技术栈**：

- 后端：Spring Boot 3.5 + MyBatis-Plus + MySQL + Redis
- 前端：Vue 3 + TypeScript + Element Plus
- 服务器：阿里云 ECS（Ubuntu 22.04）

------

## 第一步：购买云服务器

### 1.1 选择云服务商

这里以阿里云为例，新用户通常有优惠，2核2G的配置一年约99元，足够个人项目使用。完成实名认证。

### 1.2 购买配置建议

| 配置项   | 推荐选择                             |
| :------- | :----------------------------------- |
| 计费模式 | 按量付费（测试用）/ 包年包月（长期） |
| 地域     | 华北2（北京）或华东1（杭州）         |
| 实例规格 | 2核2G（经济型e）                     |
| 操作系统 | Ubuntu 22.04 64位                    |
| 公网IP   | 分配公网IPv4地址                     |
| 带宽     | 按使用流量，峰值5Mbps                |

### 1.3 配置选择指南

#### 1. 付费类型、地域、网络及可用区

![image-20260322153641363](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20260322153641363.png)

#### 2. 实例和镜像

![image-20260322153944751](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20260322153944751.png)

![image-20260322155103908](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20260322155103908.png)

![image-20260322155324691](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20260322155324691.png)

#### 3. 存储

![image-20260322155941189](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20260322155941189.png)

#### 4. 网络和安全组

![image-20260322160412443](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20260322160412443.png)

#### 5. 管理设置

![image-20260322160845961](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20260322160845961.png)

#### 6. 高级选项

![image-20260322161054361](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20260322161054361.png)



------

## 第二步：连接服务器

### 2.1 下载连接工具

推荐使用 **FinalShell**（国产免费，支持文件传输）
下载地址：http://www.hostbuf.com/t/988.html

### 2.2 连接服务器

打开 FinalShell，新建连接：

- 名称：自定义

- 主机：你的公网IP

- 端口：22

- 用户名：root

- 密码：你设置的密码

  <img src="C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20260322152036852.png" alt="image-20260322152036852" style="zoom:50%;" />

连接成功后，你会看到命令行界面。

------

## 第三步：安装运行环境

在 FinalShell 中执行以下命令（一条一条执行）：

### 3.1 更新系统

```
apt update && apt upgrade -y
```



### 3.2 安装 JDK 17

```
apt install openjdk-17-jdk -y
java -version
```



### 3.3 安装 MySQL

```
apt install mysql-server -y
systemctl start mysql
systemctl enable mysql
```



### 3.4 安装 Redis

```
apt install redis-server -y
systemctl start redis
systemctl enable redis
```



### 3.5 安装 Nginx

```
apt install nginx -y
systemctl start nginx
systemctl enable nginx
```



### 3.6 安装 Node.js（用于前端打包）

```
curl -fsSL https://deb.nodesource.com/setup_18.x | bash -
apt install nodejs -y
node -v
```

------

## 第四步：配置数据库

### 4.1 登录 MySQL

bash

```
mysql -u root -p
```

首次登录密码为空，直接回车。

### 4.2 创建数据库和用户

sql

```
CREATE DATABASE farm CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
CREATE USER 'farm'@'localhost' IDENTIFIED BY 'Farm123456!';
GRANT ALL PRIVILEGES ON farm.* TO 'farm'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```



### 4.3 导入数据（如果有备份）

```
mysql -u root -p farm < /root/farm.sql
```



------

## 第五步：打包后端项目

### 5.1 在 IDEA 中打包

打开 Maven 面板 → 找到根模块 → 双击 `clean` → 双击 `package

### 5.2 找到 jar 包

打包成功后，jar 包在：

```
farm_main/target/farm_main-0.0.1-SNAPSHOT.jar
```

<img src="C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20260322161417438.png" alt="image-20260322161417438" style="zoom:50%;" />

------

## 第六步：打包前端项目

### 6.1 修改 API 地址

创建 `.env.production` 文件：

env

```
VITE_API_BASE_URL=http://你的公网IP:8080/api
```



### 6.2 执行打包

bash

```
npm run build
```

打包后生成 `dist` 文件夹。

------

## 第七步：上传并启动后端

### 7.1 上传 jar 包

在 FinalShell 中，把 jar 包拖到 `/root` 目录。

<img src="C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20260322161639338.png" alt="image-20260322161639338" style="zoom: 50%;" />

### 7.2 启动后端

```
cd /root
nohup java -jar farm_main-0.0.1-SNAPSHOT.jar > app.log 2>&1 &
tail -f app.log
```



看到 `Started FarmMainApplication` 说明启动成功。

------

## 第八步：部署前端到 Nginx

### 8.1 上传前端文件

把 `dist` 文件夹里的所有文件上传到 `/var/www/html/` 目录。

### 8.2 验证文件

bash

```
ls -la /var/www/html/
```



应该看到 `index.html` 和 `assets` 文件夹。

------

## 第九步：配置 Nginx 反向代理

### 9.1 编辑配置文件

bash

```
vim /etc/nginx/sites-available/default
```



### 9.2 修改为以下内容

nginx

```
server {
    listen 80;
    server_name 你的公网IP;

    root /var/www/html;
    index index.html;

    location / {
        try_files $uri $uri/ /index.html;
    }

    location /api/ {
        rewrite ^/api/(.*)$ /$1 break;
        proxy_pass http://127.0.0.1:8080;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```



### 9.3 重启 Nginx

```
nginx -t
systemctl reload nginx
```



------

## 第十步：开放端口与安全组

确保阿里云安全组已开放以下端口：

| 端口 | 用途    |
| :--- | :------ |
| 22   | SSH     |
| 80   | HTTP    |
| 8080 | 后端API |

------

## 第十一步：测试访问

### 11.1 访问前端

浏览器打开 `http://你的公网IP`

### 11.2 测试登录

- 用户名：`admin`
- 密码：`admin`

### 11.3 登录页面

<img src="C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20260322163243362.png" alt="image-20260322163243362" style="zoom:50%;" />

### 11.4 查看首页

![image-20260322162957484](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20260322162957484.png)

------

## 常见问题解决

### Q1：验证码不显示？

**原因**：Redis 未启动或配置错误
**解决**：

bash

```
systemctl start redis-server
redis-cli ping  # 应返回 PONG
```



### Q2：登录报 500 错误？

**原因**：数据库连接失败
**解决**：

- 检查 MySQL 是否运行：`systemctl status mysql`
- 检查数据库名、用户名、密码是否正确

### Q3：Nginx 配置后还是显示欢迎页？

**原因**：文件未正确上传到 `/var/www/html/`
**解决**：

bash

```
ls -la /var/www/html/  # 检查是否有 index.html
```



### Q4：后端启动后立即关闭？

**原因**：端口被占用或启动参数错误
**解决**：

bash

```
pkill -9 java
nohup java -jar xxx.jar > app.log 2>&1 &
tail -f app.log
```



------

## 总结

恭喜！你已经成功将项目部署到了线上！回顾一下我们做了什么：

| 步骤 | 内容                                      |
| :--- | :---------------------------------------- |
| 1    | 购买云服务器，配置安全组                  |
| 2    | 连接服务器，安装 JDK、MySQL、Redis、Nginx |
| 3    | 创建数据库，导入数据                      |
| 4    | 打包后端 jar 包，上传并启动               |
| 5    | 打包前端 dist 文件，上传到 Nginx          |
| 6    | 配置 Nginx 反向代理                       |
| 7    | 访问公网 IP 测试                          |

**部署成功后，你的项目就可以被任何人通过公网访问了！**

------

## 📎 附录：常用命令速查

| 操作         | 命令                                       |
| :----------- | :----------------------------------------- |
| 启动后端     | `nohup java -jar xxx.jar > app.log 2>&1 &` |
| 查看后端日志 | `tail -f app.log`                          |
| 停止后端     | `pkill -f farm_main`                       |
| 重启 Nginx   | `systemctl reload nginx`                   |
| 查看端口占用 | `lsof -i:8080`                             |
| 登录 MySQL   | `mysql -u root -p`                         |

------

## 💡 写在最后

本文记录了完整的部署流程，希望对你有帮助。如果你在部署过程中遇到问题，欢迎在评论区留言交流！

**如果觉得有用，请点赞收藏支持一下～** ❤️