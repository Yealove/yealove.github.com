---
date: 2014-06-11 17:18
tags: 
- css
title: CSS3之边框发亮
---

前段时间自己鼓捣模板的时候，找到一些漂亮的实现方法。
看到一个边框发亮的非常漂亮的输入框，所以自己就拿来改造改造。下面的源码可以在任何元素中class中加入“light”就可以有边框发亮的效果（在暗色的背景下）。

```css
.light:hover {
    border-radius: 5px 5px 5px 5px;
    -webkit-animation: glow 1200ms ease-out infinite alternate;
    -moz-animation: glow 1200ms ease-out infinite alternate;
    -o-animation: glow 1200ms ease-out infinite alternate;
    -ms-animation: glow 1200ms ease-out infinite alternate;
    animation: glow 1200ms ease-out infinite alternate;

    border-color: rgba(220, 255, 220, .4);
    box-shadow: 0 0 5px rgba(220, 255, 220, .2), inset 0 0 5px rgba(220, 255, 220, .1), 0 2px 0 #000;
    outline: none;
}

@-webkit-keyframes glow {
    0% {
        border-color: rgba(220, 255, 220, .4);
        box-shadow: 0 0 5px rgba(220, 255, 220, .2), inset 0 0 5px rgba(220, 255, 220, .1)
        , 0 1px 0 rgba(220, 255, 220, .36);
    }
    100% {
        border-color: rgba(220, 255, 220, .8);
        box-shadow: 0 0 20px rgba(220, 255, 220, .3), inset 0 0 10px rgba(220, 255, 220, .34)
        , 0 1px 0 rgba(220, 255, 220, .36);
    }
}

@-moz-keyframes glow {
    0% {
        border-color: rgba(220, 255, 220, .4);
        box-shadow: 0 0 5px rgba(220, 255, 220, .2), inset 0 0 5px rgba(220, 255, 220, .1)
        , 0 1px 0 rgba(220, 255, 220, .36);
    }
    100% {
        border-color: rgba(220, 255, 220, .8);
        box-shadow: 0 0 20px rgba(220, 255, 220, .36), inset 0 0 10px rgba(220, 255, 220, .34)
        , 0 1px 0 rgba(220, 255, 220, .36);
    }
}

@-o-keyframes glow {
    0% {
        border-color: rgba(220, 255, 220, .8);
        box-shadow: 0 0 5px rgba(220, 255, 220, .2), inset 0 0 5px rgba(220, 255, 220, .1)
        , 0 1px 0 rgba(220, 255, 220, .36);
    }
    100% {
        border-color: rgba(220, 255, 220, .8);
        box-shadow: 0 0 20px rgba(220, 255, 220, .36), inset 0 0 10px rgba(220, 255, 220, .34)
        , 0 1px 0 rgba(220, 255, 220, .36);
    }
}

@-ms-keyframes glow {
    0% {
        border-color: rgba(220, 255, 220, .8);
        box-shadow: 0 0 5px rgba(220, 255, 220, .2), inset 0 0 5px rgba(220, 255, 220, .1)
        , 0 1px 0 rgba(220, 255, 220, .36);
    }
    100% {
        border-color: rgba(220, 255, 220, .8);
        box-shadow: 0 0 20px rgba(220, 255, 220, .36), inset 0 0 10px rgba(220, 255, 220, .34)
        , 0 1px 0 rgba(220, 255, 220, .36);
    }
}

@keyframes glow {
    0% {
        border-color: rgba(220, 255, 220, .8);
        box-shadow: 0 0 5px rgba(220, 255, 220, .2), inset 0 0 5px rgba(220, 255, 220, .1)
        , 0 1px 0 rgba(220, 255, 220, .36);
    }
    100% {
        border-color: rgba(220, 255, 220, .8);
        box-shadow: 0 0 20px rgba(220, 255, 220, .36), inset 0 0 10px rgba(220, 255, 220, .34)
        , 0 1px 0 rgba(220, 255, 220, .36);
    }
}
```