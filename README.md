# 前端搭建
## 搭建环境
centos 7
## 配置环境：安装lnmp1.4

1. 下载并安装LNMP一键安装包

添加虚拟主机，可根据需求，添加 ssl 支持。

```
yum install screen -y
screen -S lnmp
wget -c http://soft.vpser.net/lnmp/lnmp1.4.tar.gz && tar zxf lnmp1.4.tar.gz && cd lnmp1.4 && ./install.sh lnmp
lnmp vhost add

```

2. 修改conf

```
vim /usr/local/nginx/conf/vhost/你的域名.conf

```

添加这段到server

```
location / 
{
	try_files $uri $uri/ /index.php$is_args$args;		                
}
```

修改root行

```
root /home/wwwroot/你的域名/public;
```

示例 conf

```
server
    {
        listen 80;
        server_name 你的域名;
        return 301 https://$host$request_uri;
    }
server
    {
        listen 443 ssl;
        server_name 你的域名;
        ssl_certificate /etc/letsencrypt/live/你的域名/fullchain.pem;
        ssl_certificate_key /etc/letsencrypt/live/你的域名/privkey.pem;
        index index.html index.htm index.php default.html default.htm default.php;
        root  /home/wwwroot/你的域名/public;

        include other.conf;
        #error_page   404   /404.html;
        include enable-php.conf;

        location ~ .*\.(gif|jpg|jpeg|png|bmp|swf)$
        {
            expires      30d;
        }

        location ~ .*\.(js|css)?$
        {
			expires      12h;
        }
		location ~ /\.
		{
			deny all;
		}
location / 
		{
			try_files $uri $uri/ /index.php$is_args$args;
		}
 access_log  /home/wwwlogs/你的域名.log;
```

## 安装面板程序
### 下载面板程序

```
cd /home/wwwroot/你的域名
yum install git -y
git clone -b master https://github.com/chenmin1992/panel-v3-mod.git tmp && mv tmp/.git . && rm -rf tmp && git reset --hard
chown -R root:root *
chmod -R 755 *
chown -R www:www storage
php composer.phar install
mv tool/alipay-f2fpay vendor/
mv -f tool/autoload_classmap.php vendor/composer/
```

### 配置数据库
登陆数据库

```
mysql -u root -p          // 这里需要输入密码
mysql>use database_name   // 选择数据库
mysql>source /home/wwwroot/你的域名/sql/glzjin_all.sql  // 导入.sql文件
```

### 配置 sspanel

```
cd /home/wwwroot/你的域名
cp config/.config.php.example config/.config.php
vi config/.config.php
lnmp restart
```

着重介绍面对面支付设置部分

```
$System_Config['payment_system']='f2fpay';
 #alipay,f2fpay
$System_Config['f2fpay_app_id']='';               
$System_Config['f2fpay_p_id']='';
$System_Config['alipay_public_key']='';
$System_Config['merchant_private_key']='';
```

第一个位置： $System_Config[‘f2fpay_app_id’]=”
这个是支付宝商家平台里的APPID，开通收款码服务后进入支付宝商家平台签约管理，查看PID和Key
https://openhome.alipay.com/platform/keyManage.htm
有一个基础应用，就是那个APPID

第二个位置： $System_Config[‘f2fpay_p_id’]
这个是收款的支付宝账号，用来确认阿里消息正确性的。
https://openhome.alipay.com/platform/keyManage.htm?keyType=partner
签约管理里合作伙伴身份PID

第三个位置： $System_Config[‘alipay_public_key’]
指的是这里的支付宝公钥，注意是支付宝公钥。

第四个位置： $System_Config[‘merchant_private_key’]
这个是你自己的私钥。


### 创建管理员并同步用户

```
php xcat createAdmin          //创建管理员
php xcat syncusers            //同步用户
php -n xcat initdownload      //下载客户端
php xcat initQQWry            //下载IP解析库
php xcat resetTraffic         //重置流量
```
### 设置定时任务
执行 crontab -e命令, 添加以下五段

```
30 22 * * * php /home/wwwroot/站点文件夹/xcat sendDiaryMail 
*/1 * * * * php /home/wwwroot/站点文件夹/xcat synclogin
*/1 * * * * php /home/wwwroot/站点文件夹/xcat syncvpn
0 0 * * * php -n /home/wwwroot/站点文件夹/xcat dailyjob
*/1 * * * * php /home/wwwroot/站点文件夹/xcat checkjob    
*/1 * * * * php -n /home/wwwroot/站点文件夹/xcat syncnas
```