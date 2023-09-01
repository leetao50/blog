---
title: pointer-events介绍
date: 2023-09-01 17:30:06
tags:
---

pointer-events 属性是一个指针属性，是用于控制在什么条件下特定的图形元素可以成为指针事件的目标,当这个属性设置为none时，可以禁用 HTML 元素的 hover/focus/active 等动态效果,元素则不接收hover、click事件，由他后面的元素进行接收。



```css

/* Keyword values */
pointer-events: auto;
pointer-events: none;
pointer-events: visiblePainted; /* SVG only */
pointer-events: visibleFill;    /* SVG only */
pointer-events: visibleStroke;  /* SVG only */
pointer-events: visible;        /* SVG only */
pointer-events: painted;        /* SVG only */
pointer-events: fill;           /* SVG only */
pointer-events: stroke;         /* SVG only */
pointer-events: all;            /* SVG only */

/* Global values */
pointer-events: inherit;  从其父元素继承此属性
pointer-events: initial;  将此属性设置为其默认值

```

# 针对HTML元素

+ none：该元素永远不会成为鼠标事件的 target。但是，当其后代元素的 pointer-events 属性指定其他值时，鼠标事件可以指向后代元素，在这种情况下，鼠标事件将在捕获或冒泡阶段触发父元素的事件侦听器 (鼠标的动作将不能被该元素及其子元素所捕获，但是能够被其父元素所捕获)。
+ auto：默认值，表示指针事件已启用；此时元素会响应指针事件，阻止这些事件在其下面的元素上触发。
+ inherit：将使用 pointer-events 元素的父级的值。

除去SVG的独有属性，其他是对浏览器来说生效的属性值。
# 未设置属性pointer-events
未设置属性的情况下，在光标移动到box1可以正常的触发hover，并且移动到box1和box2重叠的部分也是触发box1的hover

![图 0](../d98836850a8653b984b300e6e5293051c1359cb67ed1f38b5efd29a38cdca6a1.gif)  

```htm
<style>
    .box1 {
      width: 100px;
      height: 100px;
      background: #04BBD4;
      margin: 20px;
      z-index: 3;
    }
    .box2 {
      width: 100px;
      height: 100px;
      background: #090A0E;
      margin: -80px 40px 20px;
      z-index: 2;
      /* pointer-events: none; */
    }
    
    .box1:hover {
      background:#078404;
    }
    
    .box2:hover {
      background:#E98889;
    }
  </style>
<div class="box1"></div>
<div class="box2"></div>
```

# 设置box1属性pointer-events为none
设置属性的情况下，在光标移动到box1无法正常的触发hover，此时hover已经失效，移动到box1和box2重叠的部分则是触发box2的hover


![图 1](../96c76495ee7c4a08932ebd3b3ba22f5f.gif)  

# 事件传播
父元素如果设置了pointer-event:none 并不意味着父元素上的事件侦听器永远不会被触发，当子元素上设置pointer-event值不是none,那么都可以通过事件传播机制来触发父元素上的事件。

该属性也可用来提高滚动时的帧频。的确，当滚动时，鼠标悬停在某些元素上，则触发其上的 hover 效果，然而这些影响通常不被用户注意，并多半导致滚动出现问题。对body元素应用pointer-events：none，禁用了包括hover在内的鼠标事件，从而提高滚动性能。

