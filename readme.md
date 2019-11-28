## 简介

本示例展示的基于函数计算 custom runtime 构建 php web 应用， 在这个案例中，可以将基于传统模式 nginx + php-fpm + mysql 开发的网站简单无缝迁移到函数计算平台。

本项目是[一元建站-基于函数计算 + wordpress 构建 serverless 网站](https://yq.aliyun.com/articles/721594)示例工程。

## 操作步骤

#### 1. clone 该工程

```bash
git clone https://github.com/awesome-fc/customruntime-php.git
```

#### 2. 安装最新版本的 fun
[fun 安装手册](https://github.com/alibaba/funcraft/blob/master/docs/usage/installation-zh.md)

在使用前，我们需要先进行配置，通过键入 fun config，然后按照提示，依次配置 Account ID、Access Key Id、Secret Access Key、 Default Region Name 即可。

#### 3. 上传网站(这里是wordpress)到远端 NAS

- template.yml 中的 NasConfig: Auto, 执行为 fun nas init

- fun nas sync， 将本地 .fun/nas/auto-default/customruntime-php/wordpress 工程上传到远端 NAS

- template.yml 中的 NasConfig 改成具体的 config（具体的 vpc 和 mnt 可以由上一步执行过程中可见）， 主要是 uid 和 gid 改成 www-data 对应的 33
 
 > 因为 fun NasConfig: Auto 默认支持的 uid 和 gid 是 10003， 所以执行函数时候，需要的用户是 www-data, 等后续 NasConfig: Auto 支持填写 uid 和 gid 功能，则不需要  NasConfig 换来换去了。
 
- 将远程 NAS 中的 wordpress 目录的所有者改成 www-data

	``` chown -R www-data:www-data wordpress ```
	
	这个可以通过 ecs 挂载 NAS 修改或者在该 service 新建一个函数，在函数中执行这个命令
	
#### 4. 执行 `fun deploy`,  成功部署service，function 和对应的自定义域名
   > 先去域名解析, 比如在示例中, 将域名 wp.mofangdegisn.cn 解析到 123456.cn-hangzhou.fc.aliyuncs.com, 对应的域名、accountId 和 region 修改成自己的

## 从头到尾 DIY

如果对本示例预先安装的 php 版本或者扩展需要有改动， 可以通过 Funfile 来定义安装软件和库， 比如 Funfile-php7.2 就是安装 php7.2 的示例

#### 1. fun 安装依赖包

如果不是有 Funfile 更新, 本工程已经下载好了， 可以 skip 掉这一步

执行 `fun install -v`, fun 会根据 Funfile 中定义的逻辑安装相关的依赖包

```
RUNTIME custom
RUN apt-get update
RUN fun-install apt-get install nginx 
RUN fun-install apt-get install php-common
RUN fun-install apt-get install php-fpm
RUN fun-install apt-get install php-cli
RUN fun-install apt-get install php-cgi
...

```

将 nginx 和 php 相关下载到本地， 这些会被下载到 .fun/root 目录下

#### 2. 更新 nginx 和 php 的 conf

比如 diff 如下: [diff](https://github.com/awesome-fc/customruntime-php/commit/b009bd7be4cb857ce6a5c7f0cbdb0f6ea4d81aa5)

主要是如下变更：

1. conf 或者扩展的路径是要正确

2. 日志设置到 NAS 的可写目录

3. 设置 nginx conf

4. 编写 bootstrap

#### 设置 bootstrap 和 nginx 的权限

```bash
chmod u+s .fun/root/usr/sbin/nginx

chmod u+s bootstrap
```

bootstrap 中有关启动命令行 config 位置可能需要根据具体情况修改

#### NEXT

按照前文中的操作步骤 3 和 4 即可
