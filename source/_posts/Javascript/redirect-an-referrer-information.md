---
title: 网页转发及referrer信息
tags: []
id: '216'
categories:
  - - JavaScript
date: 2012-09-21 23:44:19
---

测试环境: Win7 64, IE 9.0, Chrome 21.0.1180.89 m, Firefox 15.0.1 同事制作了一个导航站, 失效点任何链接,都会返回自己的主页,而相同的代码放到本地可以正确导航.由此猜测他的域名被屏蔽了. 这种屏蔽应该是利用浏览器提供的referrer信息以实现的. 只要在网页跳转时把referrer信息清空就好了. 客户端网页跳转方法 1. meta

<meta http-equiv="refresh" content="0; URL=”http://xobo.org”>

2\. javascript

window.location.href=”http://xobo.org”;
 or
window.location.replace=”http://xobo.org”;

3\. link(a)

<script>
function d(){
    document.getElementById("t").click();
}
</script>
<a id='t' href="r.php?url=index.html"  rel="noreferrer">jump</a>

方法一,方法二可以轻松在IE及Firefox中, 甩掉referrer信息. 对于Chrome来说, 真是克忠职守,不管我怎么跳转它都记录下正确的referrer信息. 还好Chrome HTML5的rel = "noreferrer"属性, 使浏览器不记录referrer值. 然而IE及Firefox又不支持"noreferrer"这么一属性.把这两种综合一个,就可以了.