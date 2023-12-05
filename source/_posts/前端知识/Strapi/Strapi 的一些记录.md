

Strapi 找回密码

[## strapi 教程, strapi 使用笔记](https://www.learnnote.site/strapi)


https://cloud.tencent.com/developer/ask/sof/106600893

我被困住了，但在Strapi论坛上找到了答案。

在项目的根目录下，运行以下命令：

```javascript
npm run strapi admin:reset-user-password --email=existing@user.com --password=NewPassword
```

复制

(您需要为您的电子邮件和密码标志使用实际值)

运行该命令后，系统将再次提示您输入相同的详细信息：

```javascript
[? User email? existing@user.com
[? New password? [hidden]
[? Do you really want to reset this user's password? Yes
```

复制

(这些需要与您在前面命令中使用的电子邮件和密码相匹配)

最后，您需要运行

```javascript
strapi build
```

复制

一旦它被重建，你将能够启动Strapi和登录到管理与你的新细节！

https://www.cnblogs.com/reamd/p/16548574.html

