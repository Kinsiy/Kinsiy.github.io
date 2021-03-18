---
title: artitalk
comments: false
keywords: 说说
date: 2021-01-17 00:01:07
description:
photos: https://cdn.jsdelivr.net/gh/Kinsiy/cdn/img/banner/shuoshuo.png
---

{% raw %}

<!-- 存放说说的容器 -->
<div id="artitalk_main"></div>
<script>
    window.addEventListener("DOMContentLoaded",()=>{
        let script = document.createElement("script");
        script.addEventListener("load", (event) => {
            new Artitalk({
                appId: "AJ8PWseL9ucLigdRhUuoDiJC-MdYXbMMI",
                appKey: "xuOLMc7TmsMi8y9BrAllkPHw",
            });
        });
        script.src = "https://cdn.jsdelivr.net/npm/artitalk";
        document.body.appendChild(script);
    });
</script>
{% endraw %}
