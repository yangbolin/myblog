title: css换行
date: 2015-05-07 21:55:59
tags: CSS
categories: 前端开发
---

最近在搞一个后台系统的时候，发现table表格的宽度没法调整，找前端大神看了一下，发现是table表格中的全英文内容过长，不会自动换行导致。因此需要用CSS来实现换行，浏览器默认不会对英文内容进行换行的。具体CSS的写法如下
```css
word-break: break-all;
word-wrap: break-word;
```
