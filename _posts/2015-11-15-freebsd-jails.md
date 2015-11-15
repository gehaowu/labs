---
layout: post
title: "将应用服务关进 Jails 以增加服务器安全性"
excerpt: "The application service shut Jails to increase server security"
modified: 2015-11-09
tags: [FreeBSD, Jail, Open Source, Service, Security]
comments: true
---

<section id="table-of-contents" class="toc">
  <header>
    <h3>全文概览</h3>
  </header>
<div id="drawer" markdown="1">
*  Auto generated table of contents
{:toc}
</div>
</section><!-- /#table-of-contents -->


### Jails 是什么？

**FreeBSD Jail** 是 *chroot* 的改进版本，几乎能提供一个完整操作系统所具有的功能。

![FreeBSD Jails]({{ site.url }}/images/2015/11/jails.png)

### 如何安装？

当前官方仍然支持的所有版本都默认支持 jail 功能，可以直接使用。

### 如何配置？

#### Jails 的主配置文件 /etc/jail.conf


{% highlight sh %}
allow.raw_sockets = 1;
exec.clean;
exec.start = "/bin/sh /etc/rc";
exec.stop = "/bin/sh /etc/rc.shutdown";
exec.consolelog = "/var/log/jail_${name}_console.log";
mount.devfs;
mount.fstab = "/etc/fstab.$name";
allow.mount;
allow.set_hostname = 0;
allow.sysvipc = 0;
path = "/usr/jails/${name}";
interface = vtnet0;
NGINX {
    jid = 1;
    host.hostname = NginxDigital.daemon.xin;
    ip4.addr = 192.168.7.1;
    ip6.addr = 0:0:0:0:0:0:C0A8:0701;
}
HEXO {
    jid = 5;
    host.hostname = HexoDigital.daemon.xin;
    ip4.addr = 192.168.9.1;
    ip6.addr = 0:0:0:0:0:0:C0A8:0901;
}
{% endhighlight %}


每个 Jails 相当于一个完整主机，所以完整系统需要的东西它也应该也有，比如 IP 地址。  

相关参数：  
**NGINX,HEXO**    他们是 Jails 配置名称，可以随意，不过得跟 rc.conf 中的匹配  
**jid**           这个是 Jails ID，XD  
**host.hostname** 是 Jails 的主机名  
**interface**     写上你的网络设备接口，你也可以直接写 *lo0* 啥的  
**ip4.addr**      你的 Jails 内网 IP，可以随意指定，比如我这里是 *192.168.7.1*  
**ip6.addr**      这个是 IPv6 的内网 IP，当然如果你没有 IPv6 支持，则可以不写  

#### 安装 FreeBSD 基本系统到 Jails

##### 获取 FreeBSD 基本系统，并释放到刚创建的目录中
{% highlight sh %}
mkdir -p mkdir /usr/jails/NGINX
fetch -o /tmp/base.txz http://ftp.freebsd.org/pub/FreeBSD/releases/amd64/10.2-RELEASE/base.txz
tar xf /tmp/base.txz -C /usr/jails/NGINX
{% endhighlight %}

##### 升级 Jails 主机基本系统
{% highlight sh %}
freebsd-update -b /usr/jails/NGINX fetch
freebsd-update -b /usr/jails/NGINX install
{% endhighlight %}

##### 配置 Jails 主机启动管理项
{% highlight sh %}
cat > /usr/jails/NGINX/etc/rc.conf <<EOF
syslogd_flags="-ssC"
clear_tmp_enable="YES"
sendmail_enable="NONE"
cron_flags="$cron_flags -J 60"
EOF
{% endhighlight %}

##### 添加 Jails 主机 DNS 服务器
{% highlight sh %}
cat > /usr/jails/NGINX/etc/resolv.conf <<EOF
nameserver 114.114.114.114
nameserver 119.29.29.29
nameserver 8.8.8.8
nameserver 8.8.4.4
EOF
{% endhighlight %}

##### 创建 fstab 文件
{% highlight sh %}
touch /etc/fstab.NGINX
{% endhighlight %}
需要挂载的项目可以直接写这里

##### 配置 PF 防火墙转发规则
{% highlight sh %}
cat >> /etc/pf.conf <<EOF
# The OpenBSD Packet Filter Configuration file
ext_if = "vtnet0"
ext_if_ipv4 = "188.166.224.72"
ext_if_ipv6 = "2400:6180:0:d0::13d:7001"
jail_webser_ipv4 = "192.168.7.1"
jail_webser_ipv6 = "::192.168.7.1"
nat on $ext_if inet from $jail_webser_ipv4 to any -> $ext_if_ipv4 round-robin
nat on $ext_if inet6 from $jail_webser_ipv6 to any -> $ext_if_ipv6 round-robin
rdr pass on $ext_if inet proto tcp from any to $ext_if_ipv4 port = http -> $jail_webser_ipv4
rdr pass on $ext_if inet proto tcp from any to $ext_if_ipv4 port = https -> $jail_webser_ipv4
rdr pass on $ext_if inet6 proto tcp from any to $ext_if_ipv6 port = http -> $jail_webser_ipv6
rdr pass on $ext_if inet6 proto tcp from any to $ext_if_ipv6 port = https -> $jail_webser_ipv6
{% endhighlight %}
因为 Jails 主机类似内网主机，所以需要母机转发相关数据到 Jails 主机，
并将 Jails 主机外发的数据转发出去。

##### 配置母机服务
{% highlight sh %}
jail_enable="YES"
jail_parallel_start="YES"
jail_list="NGINX HEXO"
pf_enable="YES"
gateway_enable="YES"
ipv6_gateway_enable="YES"
{% endhighlight %}
允许 Jails 随机启动，开启 PF 防火墙随机启动，并开启 IP 转发服务。

#### 启动 Jails 服务管理
{% highlight sh %}
service jail start
service jail stop
service jail restart
{% endhighlight %}
以上三条指令分别是启动，停止和重启 Jails 服务

### 后续管理
#### 如何登陆主机？

通过 jls 命令，列出 Jails 主机 ID ，
{% highlight sh %}
[ghw@AliyunHost.pts/1] ~/labs % jls
JID  IP Address      Hostname                      Path
1  192.168.7.1     NginxAliyun.daemon.xin        /usr/jails/NGINX
2  192.168.8.1     HoneypotAliyun.daemon.xin     /usr/jails/HONEYPOT
[ghw@AliyunHost.pts/1] ~/labs %
{% endhighlight %}

如果我要登陆 JID 为 1 的 Jails 主机，那么只需要输入
{% highlight sh %}
sudo jexec 1 tcsh
{% endhighlight %}
这样就能直接获得一个 Shell， 供您管理器 Jails 主机了， 进入主机后所有操作与本机相同。

**提示：**
试试以下命令行？
{% highlight sh %}
sudo jexec 1 service nginx restart
{% endhighlight %}

对了，您还能使用 SSH 直接远程登陆，（如果你的 Jails 主机开启了 SSH 服务）

## 没了，，就扯辣麽多！！！
![FreeBSD Jails]({{ site.url }}/images/2015/11/jails2.png)

<a href="/resources/FreeBSDHandBook/jails.html#jails-synopsis" class="btn btn-success">《FreeBSD Handbook》 Jails 概述</a>
