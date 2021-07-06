简简单单记录下踩过的坑

**前言**

在做CCNA Security lab的时候，要用802.1X和Radius服务器验证，Radius服务器搭建可以用Windows Server，为了省事（省的搭VM的时间），最简单的方法是使用 **WinRadius** 来做验证。



**交换机配置**

```cisco
! 在分配之前，Switch也要有ip地址，否则会连不到 Radius 服务器
int vlan 1
ip address 192.168.1.254 255.255.255.0
no sh

aaa new-model
radius-server host <radius-server-ip> auth-port 1812 acct-port 1813 key <per-server-key>
aaa authentication dot1x default group radius
dot1x system-auth-control 

! 注意：radius的口不要分配802.1X的验证，一般都会留最后一个口给Radius服务器用
int range fa0/1-20
switchport access vlan 10
switchport mode access
dot1x pae authenticator
exit
```



**真机测试 - Windows 7 打开802.1X验证**

win7默认是关闭802.1X的验证模式，打开方法：运行输入`services.msc` 然后找到 `Wired AutoConfig` 和 `WLAN AutoConfig` 点击启动即可，可以参考[这里](https://faq.icto.um.edu.mo/%e5%a6%82%e4%bd%95%e5%9c%a8-windows-7-%e6%93%8d%e4%bd%9c%e7%b3%bb%e7%b5%b1%e4%b8%8a%e9%80%a3%e6%8e%a5%e6%be%b3%e5%a4%a7%e6%9c%89%e7%b7%9a%e7%b6%b2%ef%bc%9f/?lang=zh-hant)。

但有时候启动还是很靠运气，上面明明显示启动了，网卡里面还是没有看到`验证`这个TAB，这个时候只能尝试把这两个服务的`Startup Type : Manual` 然后点`Stop`再点`Start`，如果还是没有的话重启下电脑，继续尝试多一次，如果真不行的话就是电脑的网卡问题。（我试过好几台机器都没有，真的挺靠运气的）

如果你的网卡有 `验证` 这个选项卡，说明你成功了，如果还是没的话，下面的都不用做了。



**真机测试 - 协议问题**

上面那一步成功了，经过了无数次测试，发现 **WinRadius** 一直提示验证错误，不支持协议，经过某人谷歌搜索后发现，原来 **WinRadius** 的旧版本都是使用 `MD5-Challenge`，因为安全的问题， window 7 默认已经不支持该选项，网上说新版的**WinRadius** 支持新的Challenge，但新版的**WinRadius**下载下来是源代码（笑），为了快速，最后还是去网上找方法看看能不能打开win7的`MD5-Challenge`



**真机测试 - 打开 windows 7 的 MD5-Challenge**

> **EAP-MD5 was removed from Windows because of its inherent lack of security. However, the MD5 functionality still exists in RASCHAP dll. You can turn on MD5 with the following steps.** 
>
> 参考：Enabling MD5-Challenge in WINDOWS

这篇文章说你可以打开注册表，加入一些关键字来打开win7的`MD5-Challenge` ，位置在` HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\RasMan\PPP\EAP` 

```Regedit
Value name: RolesSupported
Value type: REG_DWORD
Value data: 0000000a

Value name: FriendlyName
Value type: REG_SZ
Value data: MD5-Challenge

Value name: Path
Value type: REG_EXPAND_SZ
Value data: %SystemRoot%\System32\Raschap.dll

Value name: InvokeUsernameDialog
Value type: REG_DWORD
Value data: 00000001

Value name: InvokePasswordDialog
Value type: REG_DWORD
Value data: 00000001
```

我把它导出了注册表，在[这里](https://raw.githubusercontent.com/manesec/blog-open-file/master/Readme/0/005/Win7%20Enabling%20MD5-Challenge.reg)可以下载，导入了之后在验证的选项卡就会出现`MD5-Challenge` 这个选项了。



**总结**

理论和实际相差太远了，完全没考虑到这些问题。



**参考**

- [RADIUS.802.1X.配置說明](https://doomak47.pixnet.net/blog/post/25838815)
- [Enabling MD5-Challenge in WINDOWS](https://bidsarmanish.blogspot.com/2016/06/enabling-md5-challenge-in-windows_25.html?m=1)

