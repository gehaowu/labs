---
layout: post
title: "Let&rsquo;s Encrypt 开源 SSL 证书使用指南"
excerpt: "Let&rsquo;s Encrypt open source SSL Certificates Guide"
modified: 2015-11-01
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

**其实已经烂尾了，，，我已经装逼不下去了。。。。。。**


### Let&rsquo;s Encrypt 的常用指令？


获取 Let&rsquo;s Encrypt 源程序，并进入目录
{% highlight sh %}
git clone https://github.com/letsencrypt/letsencrypt
  cd letsencrypt
{% endhighlight %}


自动配置相关服务，如为NGINX/Apache添加证书相关配置。
{% highlight sh %}
./letsencrypt-auto run
{% endhighlight %}


如果没有相关支持的WEBSERVER服务，那么可以在当前目录生成证书。
{% highlight sh %}
./letsencrypt-auto -d example.com -d example.com auth
{% endhighlight %}


使用 renew 参数更新证书有效期（默认情况下会自动更新证书）
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