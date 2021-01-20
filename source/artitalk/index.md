---
title: artitalk
comments: false
keywords: 说说
date: 2021-01-17 00:01:07
description: 
photos: https://cdn.jsdelivr.net/gh/Kinsiy/cdn/img/banner/shuoshuo.png
---
{% raw %}
<script>console.log("1 "+Artitalk)</script>
<script type="text/javascript" src="https://unpkg.com/artitalk"></script>
<script>console.log("2 "+Artitalk)</script>
<!-- 存放说说的容器 -->
<div id="artitalk_main"></div>
<script type="text/javascript" > 
 try{
    var at = new Artitalk();
    at.init({
    appId: 'AJ8PWseL9ucLigdRhUuoDiJC-MdYXbMMI',
    appKey: 'xuOLMc7TmsMi8y9BrAllkPHw',
    });
}catch(e){
    window.location.reload(true);
}
</script>
{% endraw %}
