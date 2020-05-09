---
title: 单击鼠标所展示的浮动文字特效的js
date: 2018-09-12 09:11:35
tags: 
 - JS
 - 前端
categories: 
 - 前端
keywords: "前端,JS"
description: 单击鼠标所展示的浮动文字特效的js。
---

## 单击鼠标所展示的浮动文字特效的js
![](https://i.imgur.com/r3XQ6NK.gif)

```
<script type="text/javascript">
/*鼠标特效 */
  /*这个方法用来随机一个十六进制颜色代码，让每一次点击浮动文字的杨色不同*/
  function co(){
        var colorElements = "0,1,2,3,4,5,6,7,8,9,a,b,c,d,e,f";
    var colorArray = colorElements.split(",");
    var color ="#";
    for(var i =0;i<6;i++){
    color+=colorArray[Math.floor(Math.random()*16)];
    }
    return color;
};
var a_idx = 0;
jQuery(document).ready(function($) {
$("body").click(function(e) {
    /*这个数组中的每一个字符是你要浮动显示的词或句子，每次点击鼠标后按顺序出现*/
    var a = new Array("Hello基佬","你再点", "你继续点", "你有本事接着点", "有种你还点", "快别他妈点了", "好好看文章不行吗");
    var $i = $("<span/>").text(a[a_idx]);
    a_idx = (a_idx + 1) % a.length;
    var x = e.pageX,
    y = e.pageY;
    $i.css({
        "z-index": 999999999999999999999999999999999999999999999999999999999999999999999,
        "top": y - 20,
        "left": x,
        "position": "absolute",
        "font-weight": "bold",
        "color": co()
    });
    $("body").append($i);
    $i.animate({
        "top": y - 180,
        "opacity": 0
    },
    1500,
    function() {
        $i.remove();
    });
});
});
</script>
```