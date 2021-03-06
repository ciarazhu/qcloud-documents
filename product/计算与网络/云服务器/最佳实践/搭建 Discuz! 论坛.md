## 操作场景

Discuz! 是全球成熟度最高、覆盖率最大的论坛网站软件系统之一，被200多万网站用户使用。本教程介绍在 LAMP（Linux + Apache + MySQL + PHP）环境下搭建 Discuz! 论坛网站的步骤，以 Discuz! X3.2 为例。

>? 本文主要介绍自主安装 LAMP 环境并搭建 Discuz! 论坛的方法，推荐具备相关论坛搭建经验和一定的命令操作基础的用户使用。如果您第一次搭建 Discuz! 论坛且不熟悉 Linux 命令，您可以参考 [使用镜像搭建 Discuz! 论坛](https://cloud.tencent.com/document/product/213/9753)。

## 相关简介
以下是本教程中，将会使用的服务或工具：

- **云服务器**：本教程使用腾讯云云服务器（以下简称 CVM ）创建云服务器实例，来完成 WordPress 搭建工作。
- **域名注册**：如果想要使用易记的域名访问您的 WordPress 站点，可以使用腾讯云域名注册服务来购买域名。
- **网站备案**：对于域名指向中国境内服务器的网站，必须进行网站备案。在域名获得备案号之前，网站是无法开通使用的。您可以通过腾讯云 [网站备案](https://cloud.tencent.com/product/ba) 产品为您的域名备案。
- **云解析**：在配置域名解析之后，用户才能通过域名访问您的网站，而不需要使用复杂的 IP 地址。您可以通过腾讯云的 [云解析](https://cloud.tencent.com/product/cns) 服务来解析域名。

自主安装流程图如下：
![流程图2](https://main.qcloudimg.com/raw/f26817fd2a6719ccb43cbe744b0af2ed.png)

## 操作步骤

### 创建云服务器 

1. 请根据您的需要 [购买云服务器](https://buy.cloud.tencent.com/cvm?regionId=8&projectId=8)。购买指南请参考 [创建 Linux 云服务器](https://cloud.tencent.com/document/product/213/2936)
2. 服务器创建成功后，您可登录 [腾讯云管理控制台](https://console.cloud.tencent.com/cvm)  查看或编辑云服务器状态。
![实例1](https://main.qcloudimg.com/raw/891c29de005bb213fed0a7b68cfb21ff.png)

本教程中云服务器实例的操作系统版本为 CentOS 6.8。后续步骤将会用到以下信息，请注意保存：
- 云服务器实例用户名和密码。
- 云服务器实例公网 IP。

### 搭建 LAMP 环境 

对于 CentOS 系统，腾讯云提供与 CentOS 官方同步的软件安装源，包涵的软件都是当前最稳定的版本，可以直接通过 Yum 快速安装。

#### 运行 PuTTY 连接 Linux 云服务器

1. 请 [下载 PuTTY ](https://www.putty.org/) 到您的电脑，解压文件；双击 “putty.exe”，出现配置界面。
2. 选择 “Session”，在 “Host Name (or IP address)” 输入框中输入欲访问的主机名或 IP，如 “server1” 或 “192.168.2.10”。本教程输入的是云服务器实例的公网 IP。其他配置保持默认。
3. 在 “Saved Sessions” 输入栏中命名会话，单击 “Save” ，即可保存会话配置。
![putty1](//mc.qcloudimg.com/static/img/85df3247daae4982003a91ad1ad6f89e/image.png)
4. 配置完成后单击 “Open” 按钮，将会出现确认证书的提示窗，请选择 “是” 。
![putty2](//mc.qcloudimg.com/static/img/b7883110e977fb0d94310379a152c5d3/image.png)
5. 出现登录界面，依次输入云服务器实例的用户名和密码，就可连接到云服务器，进行后续操作。
![putty3](//mc.qcloudimg.com/static/img/b632cf3e122832193a77afe04c93fbc1/image.png)

<span id="InstallNecessarySoftware"></span>
#### 安装必要软件

1. 通过 PuTTY 登录云服务器后，默认已获取 root 权限，可以直接在 PuTTY 内输入命令。请输入以下命令，将必要软件一起安装 （Apache、MySQL、PHP）：
```
yum install httpd php php-fpm php-mysql mysql mysql-server -y
```
安装完成，PuTTY 窗口会提示“Complete!”。您可以上滑滚动条查看当前安装包版本：
![软件版本](//mc.qcloudimg.com/static/img/4d1e4ee237bcd67ad39051f843bded53/image.png)
本教程中安装包版本分别如下：
 - Apache：2.2.15
 - MySQL：5.1.73
 - PHP：5.3.3
2. 执行以下命令，启动服务。
>! 实际安装版本可能与示例版本有所差异。
> 如果您使用 CentOS 7 以上版本，已不再支持使用 MySQL，请替换为 MariaDB。
> 
```
service httpd start
service mysqld start
service php-fpm start
```
3. 配置 MySQL 数据库
我们需要为 Discuz! 程序创建一个独立的数据库和用户来存储数据，上一步骤已启动了数据库服务，本步骤需要给 MySQL 设定一个 root 密码，使 root 用户可以访问数据库。
```
mysqladmin -u root password "XXXXXXXX" (此处的密码可进行自定义）
```
设置好 MySQL 的密码后， 对账号密码进行验证。
```
mysql -u root -p
``` 
输入刚刚设定好的密码，可以登录到 MySQL 中，则说明配置正确。退出 MySQL：
```
exit
```
![配置数据库](//mc.qcloudimg.com/static/img/5f180f866334ee46e4b7e77851c5add0/image.png)

#### 验证环境配置

一般情况下，到此步时，环境已经配置成功，为确认和保证环境搭建成功，可以通过本步骤来验证。
1. 请使用以下命令在 在 Apache 的默认根目录 “/var/www/html” 中创建`test.php`测试文件：
```
vim /var/www/html/test.php
```
2. 按字母“I”键或 “Insert” 键切换至编辑模式，写入如下内容：
```
<?php
echo "<title>Test Page</title>";
phpinfo()
?>
```
输入完成后，按“Esc”键，输入 “:wq”，保存文件并返回。
3. 在浏览器中，访问该`test.php`文件，查看环境配置是否成功：
```
http://云服务器的公网 IP/test.php 
```
出现以下页面,则说明 LAMP 环境配置成功。
![环境验证](//mc.qcloudimg.com/static/img/3e2a1d07e4429d640461b64956b240cb/image.png)

### 配置域名（可选）

您可以给自己的 Discuz! 论坛网站设定一个单独的域名。用户可以使用易记的域名访问您的网站，而不需要使用复杂的 IP 地址。有些用户搭建论坛仅用于学习，那么可使用 IP 直接安装临时使用，但不推荐这样操作。

如果您使用 IP 直接安装，请跳过此步骤，直接进行步骤四。
如果您已有域名或者想要通过域名来访问您的论坛，请参考以下步骤。
1. 请通过腾讯云 [购买域名](https://dnspod.cloud.tencent.com/?from=qcloud)，相关域名注册指南，请参考 [域名注册](https://cloud.tencent.com/document/product/242/9595)。
2. 请进行 [网站备案](https://cloud.tencent.com/product/ba?from=qcloudHpHeaderBa&fromSource=qcloudHpHeaderBa)。
域名指向中国境内服务器的网站，必须进行网站备案。在域名获得备案号之前，网站是无法开通使用的。您可以通过腾讯云免费进行备案，一般审核时间为20天左右。
3. 通过腾讯云 [云解析](https://cloud.tencent.com/product/cns?from=qcloudHpHeaderCns&fromSource=qcloudHpHeaderCns) 配置域名解析。
 1. 登录 [云解析控制台](https://console.cloud.tencent.com/cns/domains)，选择域名或添加您已有的域名。
 2. 单击【解析】，进入该域名的域名记录管理界面。
![配置域名1](//mc.qcloudimg.com/static/img/c2e3da7449cf42697a15f5c2bf9e80cf/image.png)
 3. 单击【添加记录】，添加需要解析的记录。
![配置域名2](//mc.qcloudimg.com/static/img/4a5054890890418d83ced42db4f3a98a/image.png)

### 安装 Discuz!  

#### 下载 Discuz! 

1. 腾讯云未内置 Discuz! 安装包，请从 [Discuz! 官网](http://www.comsenz.com/downloads/install/discuzx) 下载安装包。
```
wget http://download.comsenz.com/DiscuzX/3.2/Discuz_X3.2_SC_UTF8.zip
```
2. 解压安装包。
```
unzip Discuz_X3.2_SC_UTF8.zip
```

#### 安装准备工作

1. 把解压后的 “upload” 文件夹下的所有文件复制到 “/var/www/html/”。
```
cp -r upload/* /var/www/html/
```
2. 将写权限赋予给其他用户。这些目录文件上传到服务器之后，默认只有 root 用户才有写权限。
```
chmod -R 777 /var/www/html
```

#### 安装 Discuz!

至此，论坛已经完全搭建完毕，可以在浏览器中进行安装了。
1. 在 Web 浏览器地址栏输入步骤三中配置好的域名或 Discuz! 站点的 IP 地址（云服务器实例的公网 IP 地址），可以看到 Discuz! 安装界面。单击【我同意】，进入安装步骤第一步：检查安装环境。
![安装1](//mc.qcloudimg.com/static/img/ad97b179b5b4977d86ca09a78ef05a7d/image.png)
2. 确认当前状态正常，单击 【下一步】，进入设置运行环境步骤。
![安装2](//mc.qcloudimg.com/static/img/c5a521673ed6f1a3528ba67ca5886ee4/image.png)
3. 选择全新安装，单击【下一步】，进入创建数据库步骤。
![安装3](//mc.qcloudimg.com/static/img/11a44bd86bfdfcd1fe3dcce6e8f200e6/image.png)
4. 为 Discuz! 创建一个数据库，使用 [安装必要软件](#InstallNecessarySoftware) 设置的 root 账号和密码连接数据库。并设置好系统信箱、管理员账号、密码和 Email。单击【下一步】，开始安装。
>! 请记住自己的管理员用户和密码。
>
![安装4改](//mc.qcloudimg.com/static/img/5d5184cfb34f98d791c243273b910065/image.png)
5. 安装完成后，单击【您的论坛已完成安装，点此访问】访问论坛。
![安装5](//mc.qcloudimg.com/static/img/41dab1ec86120a565bdd790238f271da/image.png)
 
