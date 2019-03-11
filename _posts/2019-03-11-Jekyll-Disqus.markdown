---
layout: post
title:  "在Jekyll中使用Disqus"
date:   2019-03-11 12:27:02 +0800
comments: true
tags:
- Jekyll
- Disqus
---
### 参考
https://blog.zxbing0066.com/jekyll/2014/11/12/disqus.html

### 注册Disqus
https://www.disqus.com/

### Add new site
进入个人主页，比如https://disqus.com/by/username/，点击右上角settings按钮， 点击Add Disqus To Site, 再次点击Get Started， 进入页面点击I want to install Disqus on my site， 进入https://disqus.com/admin/create/（也可直接进入页面）， 输入website name 和category， 点击create site, 选择basic version, 模板选择jekyll,可以看到操作提示，1️⃣再上传post的时候加入comments:true， 2️⃣在post的basic模板中， 加入如下脚本：
脚本在提示2可以直接copy。
```
<div id="disqus_thread"></div>
<script>

/**
*  RECOMMENDED CONFIGURATION VARIABLES: EDIT AND UNCOMMENT THE SECTION BELOW TO INSERT DYNAMIC VALUES FROM YOUR PLATFORM OR CMS.
*  LEARN WHY DEFINING THESE VARIABLES IS IMPORTANT: https://disqus.com/admin/universalcode/#configuration-variables*/
/*
var disqus_config = function () {
this.page.url = PAGE_URL;  // Replace PAGE_URL with your page's canonical URL variable
this.page.identifier = PAGE_IDENTIFIER; // Replace PAGE_IDENTIFIER with your page's unique identifier variable
};
*/
(function() { // DON'T EDIT BELOW THIS LINE
var d = document, s = d.createElement('script');
s.src = 'https://xxx.disqus.com/embed.js';
s.setAttribute('data-timestamp', +new Date());
(d.head || d.body).appendChild(s);
})();
</script>
<noscript>Please enable JavaScript to view the <a href="https://disqus.com/?ref_noscript">comments powered by Disqus.</a></noscript>
```
如果要显示comments的数量， 也按照提示操作即可。
