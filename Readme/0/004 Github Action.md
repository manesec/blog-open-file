临时远程桌面环境？（不算是VPS）

更新日期：2021年6月22日

**注意：请不要再用它来挖矿！！！！**

**注意：请不要再用它来挖矿！！！！**

**注意：请不要再用它来挖矿！！！！**

且行且珍惜，不然就没有了，至于会不会封号目前还不知道。


# Github Action 

GitHub Action 是 GitHub 于 2018 年 10 月推出的一个 CI\CD 服务，之前一直都是 Beta 版本，正式版于 2019 年 11 月正式推出。说白了就是你写了个自动化的脚本放到了Github的VM跑。

## 支持跑的操作系统

那么支持容器是什么操作系统呢？

| Virtual environment  | YAML workflow label                |
| :------------------- | :--------------------------------- |
| Windows Server 2019  | `windows-latest` or `windows-2019` |
| Windows Server 2016  | `windows-2016`                     |
| Ubuntu 20.04         | `ubuntu-latest` or `ubuntu-20.04`  |
| Ubuntu 18.04         | `ubuntu-18.04`                     |
| macOS Big Sur 11     | `macos-11` （仅供内部使用）        |
| macOS Catalina 10.15 | `macos-latest` or `macos-10.15`    |

另外，VM支持了哪些软件可以参考[这篇文章](https://github.com/actions/virtual-environments/tree/win19/20210616.0)。

## 检查是否有管理员权限

那么问题来了，既然可以在他的VM上跑，VM会不会给管理员权限呢？

```yaml
name: DEMO
on: workflow_dispatch
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Run a whoami
        run: whoami
      - name: Run a ID
        run: id
      - name: Run a sudo
        run: sudo whoami
```

以下是 Ubuntu 的返回结果：

```bash
> whoami
> uid=1001(runner) gid=121(docker) groups=121(docker),4(adm),101(systemd-journal)
> root
```

以下是 Mac OS 的返回结果：

```bash
> runner
> uid=501(runner) gid=20(staff) groups=20(staff),0(wheel),12(everyone),61(localaccounts),79(_appserverusr),80(admin),81(_appserveradm),98(_lpadmin),264(_webdeveloper),399(com.apple.access_ssh),701(com.apple.sharepoint.group.1),33(_appstore),100(_lpoperator),204(_developer),250(_analyticsusers),395(com.apple.access_ftp),398(com.apple.access_screensharing),400(com.apple.access_remote_ae)
> root
```

以下是 Windows 的返回结果：

```powershell
> fv-az71-869\runneradmin
> uid=197108(runneradmin) gid=197121 groups=197121
```

可以看到 Github 的 VM是有提供 Root 权限的，所以大胆猜测Windows也是有管理员权限的。

## Windows 容器

**注意**：请不要用来挖矿！！！

**大概的思路**：既然是Windows的容器，那么拿到了管理员权限就肯定可以开远程桌面，所以第一件事情就是看一下是否是公网的IP地址，如果是内网的IP地址就要使用NAT穿透。

关于NAT穿透可以使用：NGROK，花生壳，FRP等。

NGROK免费版支持一个通道。

```ymal
Windows IP Configuration


Ethernet adapter Ethernet:

   Connection-specific DNS Suffix  . : bxmsiuwdzbaetnvmcmvhpox2jg.jx.internal.cloudapp.net
   Link-local IPv6 Address . . . . . : fe80::9c2b:c01c:3d5d:cd96%13
   IPv4 Address. . . . . . . . . . . : 10.1.0.55
   Subnet Mask . . . . . . . . . . . : 255.255.0.0
   Default Gateway . . . . . . . . . : 10.1.0.1

Ethernet adapter vEthernet (nat):

   Connection-specific DNS Suffix  . : 
   Link-local IPv6 Address . . . . . : fe80::34f6:dcaf:51a8:62ab%9
   IPv4 Address. . . . . . . . . . . : 172.20.0.1
   Subnet Mask . . . . . . . . . . . : 255.255.240.0
   Default Gateway . . . . . . . . . : 
```

所以一会如果开启了RDP就要经过转发，有本地ipv6，但是没有公网ipv6。

### 创建登录管理员账号

```powershell
net config server /srvcomment:"Windows Azure VM" 
net user mane maneloveyou /add 
net localgroup administrators mane /add 
```

### 关闭密码复杂性

如果你使用弱密码，Windows 10 会自动提醒你。操作系统会提醒你的密码不符合复杂性要求。因此，系统会提示你创建新密码。

```powershell
secedit /export /cfg c:\secpol.cfg
(gc C:\secpol.cfg).replace("PasswordComplexity = 1", "PasswordComplexity = 0") | Out-File C:\secpol.cfg
secedit /configure /db c:\windows\security\local.sdb /cfg c:\secpol.cfg /areas SECURITYPOLICY
rm -force c:\secpol.cfg -confirm:$false
```

### 杂项配置

```powershell
<# 开启计算机上的物理或逻辑磁盘性能计数器 #>
diskperf -Y 
<# 开启Windows Audio服务 #>
sc start audiosrv 
sc config Audiosrv start= auto 
<# 修改指定文件的自由访问控制列表 #>
ICACLS C:\Windows\Temp /grant mane:F 
ICACLS C:\Windows\installer /grant mane:F 
<# 等待10秒应用 #>
ping -n 10 127.0.0.1 
```

### 开启RDP

```powershell
Set-ItemProperty -Path 'HKLM:\System\CurrentControlSet\Control\Terminal Server'-name "fDenyTSConnections" -Value 0
Enable-NetFirewallRule -DisplayGroup "Remote Desktop"
Set-ItemProperty -Path 'HKLM:\System\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp' -name "UserAuthentication" -Value 1
```

### 转发3389端口

这步我就不说了，市面上很多软件都可以跑。下载文件代码：

```powershell
Invoke-WebRequest xxx -OutFile xxx
```

### 虚拟锁

由于每次执行脚本之前会分配一个新的容器，每次执行完脚本就会强制关闭容器，那么在脚本的最后就需要一句话卡住脚本无限运行。

```powershell
ping 127.0.0.1 -t > null
```

然后你就可以制作脚本进行测试了，经过测试，容器在运行6个小时后会强制关机，如果要重新使用就重新运行脚本就可以了。

## Ubuntu / Google Colab

和 windows 差不多，安装vnc做个反射就可以了

# 参考

- [Windows 10: Remove Password Complexity Requirements](https://www.technipages.com/windows-10-remove-password-complexity-requirements)
- [Windows2019RDP-US](https://github.com/RixEtte/Windows2019RDP-US)

禁止转载，允许引用链接！

