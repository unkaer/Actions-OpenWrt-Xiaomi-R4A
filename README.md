# Actions-OpenWrt-Xiaomi-R4A

[![LICENSE](https://img.shields.io/github/license/mashape/apistatus.svg?style=flat-square&label=LICENSE)](https://github.com/P3TERX/Actions-OpenWrt/blob/master/LICENSE)
![GitHub Stars](https://img.shields.io/github/stars/P3TERX/Actions-OpenWrt.svg?style=flat-square&label=Stars&logo=github)
![GitHub Forks](https://img.shields.io/github/forks/P3TERX/Actions-OpenWrt.svg?style=flat-square&label=Forks&logo=github)

Build OpenWrt using GitHub Actions

[Read the details in my blog (in Chinese) | 中文教程](https://p3terx.com/archives/build-openwrt-with-github-actions.html)

# 修改说明

魔改成了
使用 `Lean` 大佬的 https://github.com/coolsnowwolf/lede openwrt源码编译 `小米R4A千兆版` 的 `breed直刷版`

1. yml中修改 openwrt源码，使其能 breed 直刷 （参考[pidge](https://www.right.com.cn/FORUM/space-uid-221258.html) 提供的方法 [分享小米R4A千兆版编译OPENWRT(Breed直刷版)](https://www.right.com.cn/FORUM/thread-4052254-1-1.html) 修改，编译生成的固件就可以用breed直刷了）
```sh
mv mt7621.mk openwrt/target/linux/ramips/image/mt7621.mk
mv mt7621_xiaomi_mir3g-v2.dts openwrt/target/linux/ramips/dts/mt7621_xiaomi_mir3g-v2.dts
```

2. min.comfig 精简了一下默认配置

```md
#去除
luci-app-accesscontrol	#访问时间控制
luci-app-arpbind		#IP/MAC绑定
luci-app-autoreboot		#支持计划重启
luci-app-ddns		#动态域名 DNS（集成阿里DDNS客户端）
luci-app-filetransfer		#文件传输（可web安装ipk包）
luci-app-nlbwmon		#网络带宽监视器
luci-app-ramfree		#释放内存
luci-app-unblockmusic	#解锁网易云灰色歌曲3合1新版本
luci-app-upnp		#通用即插即用UPnP（端口自动转发）
luci-app-vlmcsd		#KMS服务器设置
luci-app-vsftpd		#FTP服务器
luci-app-wol		#WOL网络唤醒

#保留
luci-app-firewall		#添加防火墙
luci-app-flowoffload		#Turbo ACC网络加速
luci-app-webadmin		#Web管理页面设置

#添加
luci-theme-argon		#argon主题
```

3. 修改diy-part1.sh

添加一些插件 [kenzok8/openwrt-packages](https://github.com/kenzok8/openwrt-packages) 项目

```sh
# 取消插件注释
#sed -i 's/^#\(.*helloworld\)/\1/' feeds.conf.default

# 添加插件源码
# sed -i '$a src-git ddnsto https://github.com/linkease/ddnsto-openwrt' feeds.conf.default
# sed -i '$a src-git adguardhome https://github.com/rufengsuixing/luci-app-adguardhome' feeds.conf.default
# sed -i '$a src-git dnsfilter https://github.com/garypang13/luci-app-dnsfilter' feeds.conf.default

# 添加插件源码
sed -i '$a src-git kenzo https://github.com/kenzok8/openwrt-packages' feeds.conf.default
# passwall依赖
sed -i '$a src-git small https://github.com/kenzok8/small' feeds.conf.default
```

4. 修改diy-part2.sh

修改 `diy-part2.sh` 参考自 [281677160/build-openwrt](https://github.com/281677160/build-openwrt) 

```sh
# 修改openwrt登陆地址,把下面的192.168.5.1修改成你想要的就可以了
sed -i 's/192.168.1.1/192.168.5.1/g' package/base-files/files/bin/config_generate

# 修改主机名字，把Xiaomi-R4A修改你喜欢的就行（不能纯数字或者使用中文）
sed -i '/uci commit system/i\uci set system.@system[0].hostname='Xiaomi-R4A'' package/lean/default-settings/files/zzz-default-settings

# 版本号里显示一个自己的名字（ababwnq build $(TZ=UTC-8 date "+%Y.%m.%d") @ 这些都是后增加的）
sed -i "s/OpenWrt /ababwnq build $(TZ=UTC-8 date "+%Y.%m.%d") @ OpenWrt /g" package/lean/default-settings/files/zzz-default-settings

# 修改 argon 为默认主题,可根据你喜欢的修改成其他的（不选择那些会自动改变为默认主题的主题才有效果）
sed -i 's/luci-theme-bootstrap/luci-theme-argon/g' feeds/luci/collections/luci/Makefile

# 设置密码为空（安装固件时无需密码登陆，然后自己修改想要的密码）
sed -i 's@.*CYXluq4wUazHjmCDBCqXF*@#&@g' package/lean/default-settings/files/zzz-default-settings
```
# 使用

## Fork

右上角，Fork本项目到你的仓库

## Actions

同意工作流，然后开整。

![](https://gitee.com/Unkaer/blog/raw/master/images/material/20210307205947.webp)

等到 ssh连接 界面
![](https://gitee.com/Unkaer/blog/raw/master/images/material/20210307210916.webp)

单击 `url` 进行访问;
![](https://gitee.com/Unkaer/blog/raw/master/images/material/20210307210937.webp)

黑屏 按 `Ctrl`+`C`变为命令行模式 ;
输入 `cd openwrt/ && make menuconfig` 进入菜单![](https://gitee.com/Unkaer/blog/raw/master/images/material/20210307211012.webp)
![](https://gitee.com/Unkaer/blog/raw/master/images/material/20210307211148.webp)

### 设置插件
插件对照参考 [OpenWrt 编译 LuCI -> Applications 添加插件应用说明-L大](https://www.right.com.cn/forum/thread-3682029-1-1.html)

## 3.6 下载固件
一般两个小时左右编译完成
![](https://gitee.com/Unkaer/blog/raw/master/images/material/20210307211649.webp)

## 3.7 刷入体验

附·一些图片

![](https://gitee.com/Unkaer/blog/raw/master/images/material/20210314184159.webp)

![](https://gitee.com/Unkaer/blog/raw/master/images/material/20210314184218.webp)


## Usage

- Click the [Use this template](https://github.com/P3TERX/Actions-OpenWrt/generate) button to create a new repository.
- Generate `.config` files using [Lean's OpenWrt](https://github.com/coolsnowwolf/lede) source code. ( You can change it through environment variables in the workflow file. )
- Push `.config` file to the GitHub repository.
- Select `Build OpenWrt` on the Actions page.
- Click the `Run workflow` button.
- When the build is complete, click the `Artifacts` button in the upper right corner of the Actions page to download the binaries.

## Tips

- It may take a long time to create a `.config` file and build the OpenWrt firmware. Thus, before create repository to build your own firmware, you may check out if others have already built it which meet your needs by simply [search `Actions-Openwrt` in GitHub](https://github.com/search?q=Actions-openwrt).
- Add some meta info of your built firmware (such as firmware architecture and installed packages) to your repository introduction, this will save others' time.

## Acknowledgments

- [Microsoft Azure](https://azure.microsoft.com)
- [GitHub Actions](https://github.com/features/actions)
- [OpenWrt](https://github.com/openwrt/openwrt)
- [Lean's OpenWrt](https://github.com/coolsnowwolf/lede)
- [tmate](https://github.com/tmate-io/tmate)
- [mxschmitt/action-tmate](https://github.com/mxschmitt/action-tmate)
- [csexton/debugger-action](https://github.com/csexton/debugger-action)
- [Cowtransfer](https://cowtransfer.com)
- [WeTransfer](https://wetransfer.com/)
- [Mikubill/transfer](https://github.com/Mikubill/transfer)
- [softprops/action-gh-release](https://github.com/softprops/action-gh-release)
- [ActionsRML/delete-workflow-runs](https://github.com/ActionsRML/delete-workflow-runs)
- [dev-drprasad/delete-older-releases](https://github.com/dev-drprasad/delete-older-releases)
- [peter-evans/repository-dispatch](https://github.com/peter-evans/repository-dispatch)

## License

[MIT](https://github.com/P3TERX/Actions-OpenWrt/blob/main/LICENSE) © P3TERX
