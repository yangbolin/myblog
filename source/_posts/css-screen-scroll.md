title: 实现页面区块随屏幕滚动的效果
date: 2014-04-16 18:56:55
tags: [CSS,jQuery]
categories: 前端开发
---

####一.需求描述
为了提升用户体验，我们经常需要在页面上显示一块内容，这块内容当屏幕向下滚动到一定位置后就展示出来，然后随着屏幕一起上下滚动，当屏幕向上滚动到一定程度，这块内容就消失，这样这块内容在就会有时候消失，有时候随着屏幕一起滚动。

<!-- more -->

####二.需求分析
这个需求需要从两个方面来把握，第一，内容块随着屏幕的滚动一直展示在页面上的某个位置，第二，当屏幕向上滚动到一定位置，内容块隐藏，向下滚动到一定位置，内容块显示出来，先实现第一点，在实现第二点，这个需求就满足了。

####三.实现思路
按照上面的分析，我们先来看第一点如何实现，内容块随着屏幕的滚动一直出现在页面上的某个位置，我们很容易想到css的position:fixed这个属性，可是别太高兴，IE6说你这个属性我不认识，所以在我的地盘你这样我不生效。这个时候就需要我们解决浏览器兼容性问题了，解决完这个问题我们就成功了一大半了,解决浏览器兼容性问题，使用hack啦，我们在IE6下面通过hack来使用绝对定位，伪代码如下：
```css
.mod-scroll{ 
    position:fixed; 
    top:0; 
    left:50%;
    /** for IE6, 这个表达式可以根据实际需求具体调整 **/
	_top: expression(documentElement.scrollTop + documentElement.clientHeight-this.offsetHeight);
	_position:absolute;
}
```
通过上面的css我们第一个问题就解决了，第二个问题需要监听一下滚动事件，当滚动到某个高度的时候显示内容块，当滚动到另外一个高度的时候把内容块隐藏起来。伪代码如下：
```javascript
$(window).on('scroll',function(){
    // 260这个高度可以自己根据实际需求调整
	if($(window).scrollTop() > 260) {
	    // 内容区块带有class='xxxx'
		$('div.xxxx').show();
		// 25也可以自己调整
		$('div.xxxx').css('top',($(window).scrollTop()-25)+ 'px');
	} else {
		$('div.xxx').hide();
	}
});
```

这样上面这个需求就完美解决啦。

