# xiaomi-be3600-zerotier

## 解锁

[参考文章](https://github.com/uyez/lyq/releases/tag/be3600)

## zerotier安装

1. 登录ssh在/userdisk文件夹中创建安装zerotier的文件夹，比如wp, `mkdir /userdisk/wp -p`
2. 设置安装路径为wp， 编辑 **/etc/opkg.conf** 增加 **dest wp /userdisk/wp**
3. 安装zerotier的依赖(文件下载在[package](https://downloads.openwrt.org/releases/18.06.0/packages/arm_cortex-a7_neon-vfpv4/packages/), [kmod-tun](https://downloads.openwrt.org/releases/21.02.4/targets/ipq40xx/generic/packages/))

`opkg install --force-depends -d wp kmod-tun_5.4.215-1_arm_cortex-a7_neon-vfpv4.ipk`

`opkg install -d wp libminiupnpc_2.0.20170509-1_arm_cortex-a7_neon-vfpv4.ipk`

`opkg install -d wp libnatpmp_20150609-1_arm_cortex-a7_neon-vfpv4.ipk`

4. 安装完了， 然后修改定时任务
文件是->**/data/etc/crontabs/root**
增加`* * * * * /userdisk/wp/usr/bin/start.sh`
5. 初次配置(准备zerotier配置备份文件夹)
因为安装的路径不是系统路径，库这些用不了，因此先写一个脚本用来source环境，用于我们调用zerotier

```bash
#!/bin/ash
wp_dir=/userdisk/wp
export PATH=$PATH:$wp_dir/usr/bin
export LD_LIBRARY_PATH=$wp_dir/usr/lib
```

然后`zerotier-one&`后台开启

然后`zerotier-cli info`看一下自己的id

然后`zerotier-cli join 网络号`, 然后连接ok

然后我们拷贝当前配置`cp -a /var/lib/zerotier-one /userdisk/wp/etc/`

6. 增加 **start.sh**文件

```bash
#!/bin/ash
wp_dir=/userdisk/wp
export PATH=$PATH:$wp_dir/usr/bin
export LD_LIBRARY_PATH=$wp_dir/usr/lib
# 看看zerotier-one是否在
pgrep zerotier-one
if [ $? -ne 0 ];then
        # 拷贝配置文件，保证id不变化
        ash -c "cp -a $wp_dir/etc/zerotier-one /var/lib/"
        ash -c $wp_dir/usr/bin/zerotier-one&
        # 防火墙配置，可以访问局域网
        ash -c "iptables -I FORWARD -i ztbto5vxns -j ACCEPT"
        ash -c "iptables -I FORWARD -o ztbto5vxns -j ACCEPT"
        ash -c "iptables -t nat -I POSTROUTING -o ztbto5vxns -j MASQUERADE"
        sleep 5
        # 加入网络
        ash -c "$wp_dir/usr/bin/zerotier-cli join 12ac4a1e7190c3c3"
else
        echo "zerotier-one is online"
fi
```

7. 重启，然后看连接是否ok
