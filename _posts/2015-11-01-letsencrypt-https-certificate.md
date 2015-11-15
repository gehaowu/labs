---
layout: post
title: "使用 Let&rsquo;s Encrypt 开源 SSL 证书"
excerpt: "With Let&rsquo;s Encrypt open source SSL Certificates"
modified: 2015-11-09
tags: [Let&rsquo;s&nbsp;Encrypt, Open Source, SSL]
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


### Let&rsquo;s Encrypt 是什么？

![Let&rsquo;s Encrypt]({{ site.url }}/images/2015/11/ZsUlrVAwD16p.png)
{: .image-pull-right}

Mozilla、思科、Akamai、IdenTrust、EFF 和密歇根大学研究人员联合宣布了 Let’s Encrypt CA 项目，计划为网站提供免费的基本 SSL 证书，以加速互联网从 HTTP 向 HTTPS 过渡。Let’s Encrypt CA 将由非赢利组织 Internet Security Research Group (ISRG) 运营，计划于 2015 年夏天开始向任何需要加密证书的网站自动发放免费的 SSL 证书。

**Let&rsquo;s Encrypt** 是一个将于2015年末推出的数字证书认证机构，
将通过旨在消除当前手动创建和安装证书的复杂过程的自动化流程，
为安全网站提供免费的SSL/TLS证书。

Let&rsquo;s Encrypt 是由公益组织互联网安全研究小组（ISRG）维护的。
2015年4月9日，ISRG与Linux基金会宣布合作。
它的主要赞助商包括电子前哨基金会，Mozilla基金会，Akamai以及思科。


预计将在2015年11月16日全面开始提供服务。


### Let&rsquo;s Encrypt 是如何工作的？

这个好像是重点？ 好复杂，，自己去官方看文档。。

### 申请证书 Let&rsquo;s Encrypt


获取 Let&rsquo;s Encrypt 源程序，并进入目录
{% highlight sh %}
git clone https://github.com/letsencrypt/letsencrypt
  cd letsencrypt
{% endhighlight %}

首次运行直接执行letsencrypt-auto，它会为你的系统配置好 Let&rsquo;s Encrypt 运行所需的环境  


(如果你的系统是FreeBSD，则需要多添加--debug参数)
{% highlight sh %}
./letsencrypt-auto --debug
grep: /etc/os-release: 文件或目录不存在
Bootstrapping dependencies for FreeBSD...
+ pkg install -Ay git python py27-virtualenv augeas libffi
Updating FreeBSD repository catalogue...
FreeBSD repository is up-to-date.
All repositories are up-to-date.
Updating database digests format: 100%
Checking integrity... done (0 conflicting)
The most recent version of packages are already installed
Creating virtual environment...
Updating letsencrypt and virtual environment dependencies...You are using pip version 7.1.0, however version 7.1.2 is available.
You should consider upgrading via the 'pip install --upgrade pip' command.
.You are using pip version 7.1.0, however version 7.1.2 is available.
You should consider upgrading via the 'pip install --upgrade pip' command.
...
Running with virtualenv: sudo /home/ghw/.local/share/letsencrypt/bin/letsencrypt --debug
{% endhighlight %}

环境初始化完成会弹出同意许可的窗口，你可以一步步照着来就 OK 了


我是直接使用命令行的 webroot 模式完成密钥申请的。。。窗口实在太麻烦了
{% highlight sh %}
sudo ~/.local/share/letsencrypt/bin/letsencrypt \
        --debug \
        -vvvvv \
        --text \
        --agree-dev-preview \
        --agree-tos \
        -a webroot \
        --webroot-path /usr/home/wwwroot/www.gehaowu.com \
        --server https://acme-v01.api.letsencrypt.org/directory \
        --rsa-key-size 4096 \
        --email webmaster@gehaowu.com -d www.gehaowu.com -d gehaowu.com \
        auth
{% endhighlight %}

*这里的 --debug，-vvvvv， --text，并不是必要参数，正式版发布之后，--agree-dev-preview，--server，参数也将不需要添加*

webroot-path 是你的网站根目录，

比如我有一个存放在 “/usr/home/wwwroot/www.gehaowu.com/robots.txt”

它可以通过 https://www.gehaowu.com/robots.txt 访问到，那么网站根目录就是：

/usr/home/wwwroot/www.gehaowu.com



**提示：**
webroot 模式需要为申请证书的站点指定目录添加 mime 类型，
NGINX 添加以下内容到指定站点配置文件
{% highlight sh %}
location            ~ /.well-known/acme-challenge/(.*) {
    default_type    text/plain;
    }
{% endhighlight %}

Apache 添加：
{% highlight sh %}
<IfModule mod_headers.c>
    <LocationMatch "/.well-known/acme-challenge/*">
        Header set Content-Type "application/jose+json"
    </LocationMatch>
</IfModule>
{% endhighlight %}
个人认为 webroot 模式最方便省事

Let&rsquo;s Encrypt 的服务端貌似 BUG 很多，你需要将 DNS 切换到国外的
如我是切换到 cloudflare  隔了差不多 6 个小时候之后成功申请到证书的

![Let&rsquo;s Encrypt]({{ site.url }}/images/2015/11/ghwletsencrypt.png)


## 下面的以后在写。。

自动配置相关服务，如为NGINX/Apache添加证书相关配置。
{% highlight sh %}
./letsencrypt-auto run
{% endhighlight %}


如果没有相关支持的WEBSERVER服务，那么可以在当前目录生成证书。
{% highlight sh %}
./letsencrypt-auto -d example.com -d example.com auth
{% endhighlight %}

目前使用 renew 参数更新证书有效期（默认情况下会自动更新证书）
{% highlight sh %}
./letsencrypt-auto renew --cert-path example-cert.pem
{% endhighlight %}


使用 revoke 吊销证书


吊销指定证书
{% highlight sh %}
./letsencrypt-auto revoke --cert-path example-cert.pem
{% endhighlight %}

吊销指定私钥
{% highlight sh %}
./letsencrypt-auto revoke --key-path example-key.pem
{% endhighlight %}


<a href="https://letsencrypt.org/" class="btn btn-success">Go Go Go Let&rsquo;s Encrypt Website</a>
