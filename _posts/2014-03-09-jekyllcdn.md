---
layout: post
title: "jekyll使用七牛的CDN"
description: ""
category: "jekyll"
tags: cdn
---
{% include JB/setup %}

###注册七牛

进入[七牛的用户界面](https://portal.qiniu.com/),创建一个空间,访问控制选择公开,然后点击一键加速网站,填写你的网站地址

###修改_config.xml

    safe: false
    cdn_url : http://bboniao.qiniudn.com
    JB :
    ASSET_PATH : false
    IMAGE_PATH : false

<!-- more -->
###修改_includes/JB/setup,

这样的好处本地不会使用cdn,而发布到githup上使用`site.cdn_url`上的资源

![image]({{ IMAGE_PATH }}/qiniu-cdn-jekyll-code.png)

{% if site.JB.ASSET_PATH %}
  {% assign ASSET_PATH = site.JB.ASSET_PATH %}
{% elsif site.safe %}
  {% capture ASSET_PATH %}{{ site.cdn_url }}/assets/themes/{{ page.theme.name }}{% endcapture %}
{% else %}
  {% capture ASSET_PATH %}{{ BASE_PATH }}/assets/themes/{{ page.theme.name }}{% endcapture %}
{% endif %}

{% if site.JB.IMAGE_PATH %}
  {% assign IMAGE_PATH = site.JB.IMAGE_PATH %}
{% elsif site.safe %}
  {% capture IMAGE_PATH %}{{ site.cdn_url }}/images{% endcapture %}
{% else %}
  {% capture IMAGE_PATH %}{{ BASE_PATH }}/images{% endcapture %}
{% endif %}

###说明

/assets和/images下的资源会缓存到cdn,使用的时候用 \{\{ ASSET_PATH \}\} 和 \{\{ IMAGE_PATH \}\} 来代替.更改缓存的内容时,需要到七牛后台界面的空间设置--高级设置--去刷新
